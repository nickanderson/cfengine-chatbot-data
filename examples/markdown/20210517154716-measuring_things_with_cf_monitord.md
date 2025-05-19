CFEngine\'s [measurement
promises](https://docs.cfengine.com/docs/lts/reference-promise-types-measurements.html)
let you track numeric values by extracting values from a file or the
output of a command,
[`cf-monitord`](https://docs.cfengine.com/docs/lts/reference-components-cf-monitord.html),
the component responsible for *measurement* type promises stores these
values in the `mon`{.verbatim} bundle scope, named for the *promise
handle* and can automatically define classes when the last measured
value is 1, 2, or 3 standard deviations higher or lower than the average
which can be used for decision making in other policies.

# Measuring the number of files in a directory. {#2d03f234-00dc-4b02-bb4f-b1d06b8c515d created="[2021-05-17 Mon 15:52]"}

First, let\'s build a policy to provide a value that we can track over
time.

:::: captioned-content
::: caption
Example Policy
:::

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both"}
bundle agent __main__
{
    methods:
      "Record count of files in directory for cf-monitord to measure"
        usebundle => count_files_in_directory; 

    reports:

    # Just for illustration

      "Files in directory: $(count_files_in_directory._num)"
        if => isvariable( "count_files_in_directory._num" );

      "No directory with contents to count!"
        unless => isdir( "$(count_files_in_directory._dir)" );

      "Unable to resolve $(count_files_in_directory._num)"
        unless => isvariable( "count_files_in_directory._num" );

      "Semaphore '$(count_files_in_directory._dir)' contains:"
        printfile => cat( "$(count_files_in_directory._dir)" ),
        if => isplain( $(_count_files_in_directory._semaphore) );

}
bundle agent count_files_in_directory
{
  vars:  

    # This is the directory where we will count files
    "_dir" string => "/tmp/count_my_files"; 

    # This is where we will store the value that cf-monitord will track over time
    "_semaphore" string => "/tmp/count_my_files.dat";

    # Here we figure out how many files are in the directory
    "_num"
      int => length( lsdir( $(_dir),
                          "^(?!(\.$|\.\.$)).*", # Filter out current and parent directories
                          "false") ),
      if => isdir( $(_dir) );

 files:

    # Here we record our findings for cf-monitord to measure.
    "$(_semaphore)"
      create => "true", # Make sure this file exists
      template_method => "inline_mustache", # Render the contents of the file using an inline mustache template
      edit_template_string => "$(_num)", # We just expanded a CFEngine variable instead
      template_data => '{}'; # We used CFEnigne variable expansion, no data needed
}   
```
::::

``` example
    info: Updated rendering of '/tmp/count_my_files.dat' from mustache template 'inline'
    info: files promise '/tmp/count_my_files.dat' repaired
R: Files in directory: 1
```

Let\'s see what happens when we run the policy:

``` example
R: No directory with contents to count!
R: Unable to resolve $(count_files_in_directory._num)
```

Well, no directory, let\'s create it and add some files.

:::: captioned-content
::: caption
Shell command
:::

``` {.bash org-language="sh" results="output" exports="both"}
mkdir -p /tmp/count_my_files
touch /tmp/count_my_files/one
```
::::

Now, what do we see when we run the policy again?

``` example
    info: Updated rendering of '/tmp/count_my_files.dat' from mustache template 'inline'
    info: files promise '/tmp/count_my_files.dat' repaired
R: Files in directory: 1
```

We can see from the output that there is one file in the directory, as
expected.

Now, let\'s write a measurement promise to track this over time.

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both" tangle="/tmp/measurement-example.cf"}
bundle agent __main__
{
    methods:
      "Record count of files in directory for cf-monitord to measure"
        usebundle => count_files_in_directory; 

    reports:

    # Just for illustration

      "Files in directory: $(count_files_in_directory._num)"
        if => isvariable( "count_files_in_directory._num" );

      "No directory with contents to count!"
        unless => isdir( "$(count_files_in_directory._dir)" );

      "Unable to resolve $(count_files_in_directory._num)"
        unless => isvariable( "count_files_in_directory._num" );

      "Semaphore '$(count_files_in_directory._dir)' contains:"
        printfile => cat( "$(count_files_in_directory._dir)" ),
        if => isplain( $(_count_files_in_directory._semaphore) );
}
bundle agent count_files_in_directory
{
  vars:

    # This is the directory where we will count files
    "_dir" string => "/tmp/count_my_files";

    # This is where we will store the value that cf-monitord will track over time
    "_semaphore" string => "/tmp/count_my_files.dat";

    # Here we figure out how many files are in the directory
    "_num"
      int => length( lsdir( $(_dir),
                          "^(?!(\.$|\.\.$)).*", # Filter out current and parent directories
                          "false") ),
      if => isdir( $(_dir) );

 files:

    # Here we record our findings for cf-monitord to measure.
    "$(_semaphore)"
      create => "true", # Make sure this file exists
      template_method => "inline_mustache", # Render the contents of the file using an inline mustache template
      edit_template_string => "$(_num)", # We just expanded a CFEngine variable instead
      template_data => '{}'; # We used CFEnigne variable expansion, no data needed
}
bundle monitor measure_count_files_in_directory
{
  measurements:
      "/tmp/count_my_files.dat"
        handle => "my_count_of_files_in_directory", # Note: the handle is
                                                    # important, this maps to
                                                    # the variable that
                                                    # cf-monitord will store the
                                                    # last and average measured
                                                    # value as well as the
                                                    # standard deviation for the
                                                    # value.
        stream_type => "file",    # We get the data by parsing a file
        data_type => "int",       # We are extracting a raw whole number value
        history_type => "weekly", # This is the history type that will store the
                                  # values observed over a two dimensional time
                                  # average
        units => "files", # This describes the units of our measurement
        match_value => single_value( "\d+" ),
        comment => concat( "The number of files in the directory ", # A description of the measurement
                           "'/tmp/count-my-files'"), # Be careful that it must
                                                     # not include newlines, use
                                                     # concat() to break long
                                                     # comments over multiple
                                                     # lines for readability in
                                                     # policy
        if => fileexists( "/tmp/count_my_files.dat");
}

## Copied from the standard library ##

body match_value single_value(regex)
# @brief Extract lines matching `regex` as values
# @param regex Regular expression matching lines and values
#
# **See also:** `select_line_matching`, `extraction_regex`
{
        select_line_matching => "$(regex)";
        extraction_regex => "($(regex))";
}
```

Now, I want to check that `cf-monitord` is able to measure the value we
stored. The best *direct* way to see this is by running the policy with
`cf-monitord` in the foreground (`-F`{.verbatim}) and *verbose* logging
(`-v`{.verbatim}). Here I ran `cf-monitord` using `timeout` so that it
stops after three seconds and I grepped the output for the handle of the
measurement promise (`my_count_of_files_in_directory`{.verbatim}).

``` example
timeout 3 cf-monitord -Fv /tmp/measurement-example.cf | grep -C 3 my_count_of_files_in_directory 

 verbose: C: discovered hard class mac_54_ee_75_5a_45_41
 verbose: C: discovered hard class mem_free_high_normal
 verbose: C: discovered hard class monitor
 verbose: C: discovered hard class my_count_of_files_in_directory_high
 verbose: C: discovered hard class net_iface_docker0
 verbose: C: discovered hard class net_iface_eth0
 verbose: C: discovered hard class net_iface_lo
--
cf-monitord  verbose: Using slot ob_spare+8 (80) for mem_freeswap
cf-monitord  verbose: Gathering data from static Nova monitoring probes is finished.
cf-monitord  verbose: P: .........................................................
cf-monitord  verbose: P: BEGIN promise 'my_count_of_files_in_directory' of type "measurements" (pass 0)
cf-monitord  verbose: P:    Promiser/affected object: '/tmp/count_my_files.dat'
cf-monitord  verbose: P:    Part of bundle: measure_count_files_in_directory
cf-monitord  verbose: P:    Base context class: any
--
cf-monitord  verbose: P:    Stack path: /default/measure_count_files_in_directory/measurements/'/tmp/count_my_files.dat'[1]
cf-monitord  verbose: P:
cf-monitord  verbose: P:    Comment:  The number of files in the directory '/tmp/count-my-files'
cf-monitord  verbose: Considering promise "my_count_of_files_in_directory"
cf-monitord  verbose: Promise 'my_count_of_files_in_directory' is numerical in nature
cf-monitord     info: Sampling '/tmp/count_my_files.dat' ...(timeout=0,owner=0,group=0)
cf-monitord  verbose: Stream "/tmp/count_my_files.dat" is a plain file
cf-monitord     info: Sampling => 1
cf-monitord  verbose: Saving state for my_count_of_files_in_directory/tmp/count_my_files.dat at 1
cf-monitord     info: Collected sample of /tmp/count_my_files.dat
cf-monitord  verbose: Using slot ob_spare+12 (84) for my_count_of_files_in_directory
cf-monitord  verbose:   Found regex '\d+' matches line '1'
cf-monitord  verbose: Found candidate match value of '1'
cf-monitord     info: Extracted value "1.000000" for promise "my_count_of_files_in_directory"
cf-monitord  verbose: Setting Nova slot 84=my_count_of_files_in_directory to 1.000000
cf-monitord  verbose: CFEngine Core 3.18.0a.794e723a8 - ready
cf-monitord  verbose: ----------------------------------------------------------------
cf-monitord  verbose:  Environment discovery 
--
cf-monitord  verbose: [82] = 0.000000 -> (0.000000#0.000000) local [0.000000#0.000000]
cf-monitord  verbose: [83] cfapache_application_log_errors_count q=0.000000, var=0.000000, ex=0.000000
cf-monitord  verbose: [83] = 0.000000 -> (0.000000#0.000000) local [0.000000#0.000000]
cf-monitord  verbose: [84] my_count_of_files_in_directory q=1.000000, var=0.160000, ex=0.840000
cf-monitord  verbose: [84] = 1.000000 -> (0.840000#0.400000) local [0.504000#0.309839]
cf-monitord  verbose: Storing 1.00 in my_count_of_files_in_directory
cf-monitord  verbose: [85] spare q=0.000000, var=0.000000, ex=0.000000
cf-monitord  verbose: [85] = 0.000000 -> (0.000000#0.000000) local [0.000000#0.000000]
cf-monitord  verbose: [86] spare q=0.000000, var=0.000000, ex=0.000000
zsh: exit 124    timeout 3 cf-monitord -Fv /tmp/measurement-example.cf | 
zsh: terminated  grep -C 3 my_count_of_files_in_directory

```

In the above example we can see that the class
`my_count_of_files_in_directory_high`{.verbatim} was defined. Not really
a surprise here, we don\'t have any history so it\'s considered *high*.
Then, we can see some of the logging around the promise to sample the
data and we can see that it successfully stored the value.

Let\'s toss some more files in the directory, run the policy, and make
sure that cf-monitord picks up the new values.

``` {.bash org-language="sh" results="output" exports="both"}
touch /tmp/count_my_files/one
touch /tmp/count_my_files/two
touch /tmp/count_my_files/three
touch /tmp/count_my_files/four
```

I have added in some additional reports to the policy showing how you
can access the last, and average measured values, as well as the
standard deviation.

Running the policy (with `cf-agent`) to update the information that
`cf-monitord` will track we see:

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both" tangle="/tmp/measurement-example.cf"}
bundle agent __main__
{
    methods:
      "Record count of files in directory for cf-monitord to measure"
        usebundle => count_files_in_directory; 

    reports:

    # Just for illustration

      "Files in directory: $(count_files_in_directory._num)"
        if => isvariable( "count_files_in_directory._num" );

      "No directory with contents to count!"
        unless => isdir( "$(count_files_in_directory._dir)" );

      "Unable to resolve $(count_files_in_directory._num)"
        unless => isvariable( "count_files_in_directory._num" );

      "Semaphore '$(count_files_in_directory._dir)' contains:"
        printfile => cat( "$(count_files_in_directory._dir)" ),
        if => isplain( $(_count_files_in_directory._semaphore) );


      "The Last measured count was $(mon.value_my_count_of_files_in_directory)";
      "The Average measured count was $(mon.av_my_count_of_files_in_directory)";
      "The Standard deviation for the count is $(mon.dev_my_count_of_files_in_directory)";
}
bundle agent count_files_in_directory
{
  vars:

    # This is the directory where we will count files
    "_dir" string => "/tmp/count_my_files";

    # This is where we will store the value that cf-monitord will track over time
    "_semaphore" string => "/tmp/count_my_files.dat";

    # Here we figure out how many files are in the directory
    "_num"
      int => length( lsdir( $(_dir),
                          "^(?!(\.$|\.\.$)).*", # Filter out current and parent directories
                          "false") ),
      if => isdir( $(_dir) );

 files:

    # Here we record our findings for cf-monitord to measure.
    "$(_semaphore)"
      create => "true", # Make sure this file exists
      template_method => "inline_mustache", # Render the contents of the file using an inline mustache template
      edit_template_string => "$(_num)", # We just expanded a CFEngine variable instead
      template_data => '{}'; # We used CFEnigne variable expansion, no data needed
}
bundle monitor measure_count_files_in_directory
{
  measurements:
      "/tmp/count_my_files.dat"
        handle => "my_count_of_files_in_directory", # Note: the handle is
                                                    # important, this maps to
                                                    # the variable that
                                                    # cf-monitord will store the
                                                    # last and average measured
                                                    # value as well as the
                                                    # standard deviation for the
                                                    # value.
        stream_type => "file",    # We get the data by parsing a file
        data_type => "int",       # We are extracting a raw whole number value
        history_type => "weekly", # This is the history type that will store the
                                  # values observed over a two dimensional time
                                  # average
        units => "files", # This describes the units of our measurement
        match_value => single_value( "\d+" ),
        comment => concat( "The number of files in the directory ", # A description of the measurement
                           "'/tmp/count-my-files'"), # Be careful that it must
                                                     # not include newlines, use
                                                     # concat() to break long
                                                     # comments over multiple
                                                     # lines for readability in
                                                     # policy
        if => fileexists( "/tmp/count_my_files.dat");
}

## Copied from the standard library ##

# body match_value single_value(regex)
# # @brief Extract lines matching `regex` as values
# # @param regex Regular expression matching lines and values
# #
# # **See also:** `select_line_matching`, `extraction_regex`
# {
#         select_line_matching => "$(regex)";
#         extraction_regex => "($(regex))";
# }
```

``` example
    info: Updated rendering of '/tmp/count_my_files.dat' from mustache template 'inline'
    info: files promise '/tmp/count_my_files.dat' repaired
R: Files in directory: 4
R: The Last measured count was 1.00
R: The Average measured count was 0.84
R: The Standard deviation for the count is 0.51
```

Now, we can run cf-monitord again and we can see that it updated the
value.

``` example
timeout 3 cf-monitord -Fv /tmp/measurement-example.cf | grep -C 3 my_count_of_files_in_directory

 verbose: C: discovered hard class mac_54_ee_75_5a_45_41
 verbose: C: discovered hard class mem_free_high_dev1
 verbose: C: discovered hard class monitor
 verbose: C: discovered hard class my_count_of_files_in_directory_high
 verbose: C: discovered hard class net_iface_docker0
 verbose: C: discovered hard class net_iface_eth0
 verbose: C: discovered hard class net_iface_lo
--
cf-monitord  verbose: Using slot ob_spare+8 (80) for mem_freeswap
cf-monitord  verbose: Gathering data from static Nova monitoring probes is finished.
cf-monitord  verbose: P: .........................................................
cf-monitord  verbose: P: BEGIN promise 'my_count_of_files_in_directory' of type "measurements" (pass 0)
cf-monitord  verbose: P:    Promiser/affected object: '/tmp/count_my_files.dat'
cf-monitord  verbose: P:    Part of bundle: measure_count_files_in_directory
cf-monitord  verbose: P:    Base context class: any
--
cf-monitord  verbose: P:    Stack path: /default/measure_count_files_in_directory/measurements/'/tmp/count_my_files.dat'[1]
cf-monitord  verbose: P:
cf-monitord  verbose: P:    Comment:  The number of files in the directory '/tmp/count-my-files'
cf-monitord  verbose: Considering promise "my_count_of_files_in_directory"
cf-monitord  verbose: Promise 'my_count_of_files_in_directory' is numerical in nature
cf-monitord     info: Sampling '/tmp/count_my_files.dat' ...(timeout=0,owner=0,group=0)
cf-monitord  verbose: Stream "/tmp/count_my_files.dat" is a plain file
cf-monitord     info: Sampling => 4
cf-monitord  verbose: Saving state for my_count_of_files_in_directory/tmp/count_my_files.dat at 1
cf-monitord     info: Collected sample of /tmp/count_my_files.dat
cf-monitord  verbose: Using slot ob_spare+12 (84) for my_count_of_files_in_directory
cf-monitord  verbose:   Found regex '\d+' matches line '4'
cf-monitord  verbose: Found candidate match value of '4'
cf-monitord     info: Extracted value "4.000000" for promise "my_count_of_files_in_directory"
cf-monitord  verbose: Setting Nova slot 84=my_count_of_files_in_directory to 4.000000
cf-monitord  verbose: CFEngine Core 3.18.0a.794e723a8 - ready
cf-monitord  verbose: ----------------------------------------------------------------
cf-monitord  verbose:  Environment discovery 
--
cf-monitord  verbose: [82] = 0.000000 -> (0.000000#0.000000) local [0.000000#0.000000]
cf-monitord  verbose: [83] cfapache_application_log_errors_count q=0.000000, var=0.000000, ex=0.000000
cf-monitord  verbose: [83] = 0.000000 -> (0.000000#0.000000) local [0.000000#0.000000]
cf-monitord  verbose: [84] my_count_of_files_in_directory q=4.000000, var=9.600000, ex=2.400000
cf-monitord  verbose: [84] = 4.000000 -> (2.400000#3.098387) local [1.440000#2.400000]
cf-monitord  verbose: Storing 4.00 in my_count_of_files_in_directory
cf-monitord  verbose: [85] spare q=0.000000, var=0.000000, ex=0.000000
cf-monitord  verbose: [85] = 0.000000 -> (0.000000#0.000000) local [0.000000#0.000000]
cf-monitord  verbose: [86] spare q=0.000000, var=0.000000, ex=0.000000
zsh: exit 124    timeout 3 cf-monitord -Fv /tmp/measurement-example.cf | 
zsh: terminated  grep -C 3 my_count_of_files_in_directory

```

Updated values mean that our averages and standard deviations will
change:

```{=org}
#+RESULTS:
```
``` example
R: Files in directory: 4
R: The Last measured count was 4.00
R: The Average measured count was 2.40
R: The Standard deviation for the count is 3.92
```
