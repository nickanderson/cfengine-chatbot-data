``` cfengine3
bundle agent main
{
      # CFEngine
      # is the agent you're looking for.

  vars:

      "this_policy_file_content"
        string => readfile( $(this.promise_filename), inf ),
        if => not( isvariable( $(this.promiser) ) );

  classes:

      "I_found_the_agent_i_am_looking_for"
        expression => regcmp( $(this_policy_file_content),
                              ".*CFEngine.*is\sthe\sagent\s.*");

  reports:

      "I found the agent i am looking for"
        if => "I_found_the_agent_i_am_looking_for";

}


```

:::: captioned-content
::: caption
Example showing how to search for a mutliline pattern in a file using
native functions
:::

``` {#Example showing how to search for a mutliline pattern in a file using native functions .cfengine3}
bundle agent main
{
      # CFEngine
      # is the agent you're looking for.

  files:
    "/tmp/example.txt"
      create => "true",
      edit_line => insert_lines( "CFEngine$(const.n)is the agent you're looking for" ); 

  vars:

      "file_content"
        string => readfile( "/tmp/example.txt" , inf ),
        if => not( isvariable( $(this.promiser) ) );

  classes:

      # Search for a multi-line string
      "I_found_the_agent_i_am_looking_for"
        expression => regcmp( "CFEngine\Ris\sthe\sagent.*",
                              $(file_content));

      "I_did_not_find_droids"
        expression => not( regcmp( ".*droid.*",
                              $(file_content)));


  reports:

      "CFEngine $(sys.cf_version)";

      "/tmp/example.txt contains:$(const.n)$(file_content)";

      "I found the agent i'm looking for."
        if => "I_found_the_agent_i_am_looking_for";

      "I see no droids."
        if => "I_did_not_find_droids";
}
```
::::

```{=org}
#+RESULTS: Example showing how to search for a mutliline pattern in a file using native functions
```
``` example
R: CFEngine 3.11.0
R: /tmp/example.txt contains:
CFEngine
is the agent you're looking for

R: I found the agent i'm looking for.
R: I see no droids.
```

Aleksey Tsalolikhin \<aleksey@verticalsysadmin.com\> writes:

\> Yeah, I tried regcmp() but \"it didn\'t work\". I\'ll try again. Now
I know \> I\'m on the right track. Thanks, Nick!

\> Is there any way to do a multiline regex check for a file in
CFEngine?

I was able to search for a multi-line string using `regcmp()`{.verbatim}
and `readfile()`{.verbatim}.

It would probably be a nice addition to have a function that could do
grep natively and return boolean to define a class. For this case at
least then you wouldn\'t need to keep the entire file content stored in
a variable.

We have a grep function, but it operates on lists and not files.
<http://docs.cfengine.com/docs/master/reference-functions-grep.html>

:::: captioned-content
::: caption
Example showing how to search for a mutliline pattern in a file using
native functions
:::

``` {#Example showing how to search for a mutliline pattern in a file using native functions .cfengine3}
bundle agent main
{
      # CFEngine
      # is the agent you're looking for.

  files:
    "/tmp/example.txt"
      create => "true",
      edit_line => insert_lines( "CFEngine$(const.n)is the agent you're looking for" );

  vars:

      "file_content"
        string => readfile( "/tmp/example.txt" , inf ),
        if => not( isvariable( $(this.promiser) ) );

  classes:

      # Search for a multi-line string
      "I_found_the_agent_i_am_looking_for"
        expression => regcmp( "CFEngine\Ris\sthe\sagent.*",
                              $(file_content));

      "I_did_not_find_droids"
        expression => not( regcmp( ".*droid.*",
                              $(file_content)));


  reports:

      "CFEngine $(sys.cf_version)";

      "/tmp/example.txt contains:$(const.n)$(file_content)";

      "I found the agent i'm looking for."
        if => "I_found_the_agent_i_am_looking_for";

      "I see no droids."
        if => "I_did_not_find_droids";
}
```
::::

Results in this output:

```{=org}
#+RESULTS: Example showing how to search for a mutliline pattern in a file using native functions
```
``` example
R: CFEngine 3.11.0
R: /tmp/example.txt contains:
CFEngine
is the agent you're looking for

R: I found the agent i'm looking for.
R: I see no droids.
```
