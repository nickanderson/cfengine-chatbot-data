``` {.cfengine3 tangle="dynamic_body_action_action_policy.cf"}
bundle agent __main__
{
  vars:

      # Set this variable to "warn" if /tmp/file-action-warn.fix does not exist
      "tmp_file_action_warn_action_policy"
        string => "warn",
        unless => fileexists( "/tmp/file-action-warn.fix" );

      # Set this variable to "fix" if /tmp/file-action-warn.fix contains a class
      # expression that evaulates true, if it doesn't evaluate to true, then "warn"
      "tmp_file_action_warn_action_policy"
        string => ifelse ( readfile( "/tmp/file-action-warn.fix" ), "fix",
                           "warn"),
        if => fileexists( "/tmp/file-action-warn.fix" );

  files:
      "/tmp/file-action-default"
        create => "true",
        content => "Hello world from /tmp/file-action-default";

      "/tmp/file-action-warn"
        create => "true",
        content => "Hello world from /tmp/file-action-warn",
        action => policy( "$(tmp_file_action_warn_action_policy)" );
}

body action policy( action_policy )
{
        action_policy => "$(action_policy)";
}
```

Currently, there is no `/tmp/file-action-warn.fix`{.verbatim} file to
take a signal from, so we will default to warn.

``` {.bash org-language="sh" results="output" exports="both"}
exec 2>&1
rm -f /tmp/file-action-warn /tmp/file-action-default
cf-agent --no-lock --log-level inform --file ./body_action_action_policy.cf
:
```

``` example
    info: Created file '/tmp/file-action-default', mode 0600
    info: Updated content of '/tmp/file-action-default' with content 'Hello world from /tmp/file-action-default'
    info: files promise '/tmp/file-action-default' repaired
 warning: Should create file '/tmp/file-action-warn', mode '0600', but only warning promised
 warning: Warnings encountered when actuating files promise '/tmp/file-action-warn'
```

Now, let\'s create the file that we can take a signal from. Let\'s say
that this file is allowed to be fixed today, I populated the file with
an expression that is true.

``` {.bash org-language="sh" results="output" exports="both"}
exec 2>&1
rm -f /tmp/file-action-warn /tmp/file-action-default
echo "Tuesday.April.Day6.Yr2021" > /tmp/file-action-warn.fix
cf-agent --no-lock --log-level inform --file ./body_action_action_policy.cf
:
```

Notice in the output that the file was modified as expected.

```{=org}
#+RESULTS:
```
``` example
    info: Created file '/tmp/file-action-default', mode 0600
    info: Updated content of '/tmp/file-action-default' with content 'Hello world from /tmp/file-action-default'
    info: files promise '/tmp/file-action-default' repaired
    info: Created file '/tmp/file-action-warn', mode 0600
    info: Updated content of '/tmp/file-action-warn' with content 'Hello world from /tmp/file-action-warn'
    info: files promise '/tmp/file-action-warn' repaired
```

And, what if we put an expression in that evaluates false?

``` {.bash org-language="sh" results="output" exports="both"}
exec 2>&1
rm -f /tmp/file-action-warn /tmp/file-action-default
echo "Friday.April.Day9.Yr2021" > /tmp/file-action-warn.fix
cf-agent --no-lock --log-level inform --file ./body_action_action_policy.cf
:
```

Notice the output indicates the file was not modified.

```{=org}
#+RESULTS:
```
``` example
    info: Created file '/tmp/file-action-default', mode 0600
    info: Updated content of '/tmp/file-action-default' with content 'Hello world from /tmp/file-action-default'
    info: files promise '/tmp/file-action-default' repaired
 warning: Should create file '/tmp/file-action-warn', mode '0600', but only warning promised
 warning: Warnings encountered when actuating files promise '/tmp/file-action-warn'
```
