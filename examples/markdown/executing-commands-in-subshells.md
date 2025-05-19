This was [originally published
here](https://cmdln.org/2019/03/08/executing-commands-with-substitution-from-cfengine/),
it has been re-published with permission.

> How can I execute a command that uses command substitution in
> CFEngine?

On the console I might execute something like this:

:::: captioned-content
::: caption
Example command substitution
:::

``` {.bash org-language="sh" exports="both" results="output"}
touch /tmp/file-$(date --iso-8601) 
ls /tmp/file-*
```
::::

``` example
/tmp/file-2019-03-08
```

I recommend not executing commands using substitution. Instead, prepare
all that you need up front. Get the result of the data command and put
it into a cfengine variable, then use the cfengine variable directly.

``` {.cfengine3 include-stdlib="t" verbose-mode="nil" inform-mode="nil" exports="both" results="output"}
bundle agent main
{
  vars:
      "date" string => execresult( "date --iso-8601=second", useshell );
      "command" string => "/usr/bin/touch /tmp/file-$(date)";
      "result" string => execresult( $(command), useshell );

  reports:
      "CFEngine $(sys.cf_version)";
      "Executed $(command)" if => isvariable( result );
      "Files:$(const.n)$(with)" with => join( "$(const.n)", lsdir( "/tmp/", "file-.*", false ));
}
```

``` example
R: CFEngine 3.13.0
R: Executed /usr/bin/touch /tmp/file-2019-03-08T11:34:37-06:00
R: Files:
file-2019-03-08T11:34:21-06:00    
file-2019-03-08T11:34:34-06:00    
file-2019-03-08T11:34:37-06:00    
file-2019-03-08T11:34:24-06:00
```

But, if you really want to use command substitution you can use
backticks directly.

``` {.cfengine3 include-stdlib="t" verbose-mode="nil" inform-mode="nil" exports="both" results="output"}
bundle agent main
{
  vars:
      "command" string => "/usr/bin/touch /tmp/file2-`date --iso-8601=second`";
      "result" string => execresult( $(command), useshell );

  reports:
      "CFEngine $(sys.cf_version)";
      "Executed $(command)" if => isvariable( result );
      "Files:$(const.n)$(with)" with => join( "$(const.n)", lsdir( "/tmp/", "file2-.*", false ));
}
```

``` example
R: CFEngine 3.13.0
R: Executed /usr/bin/touch /tmp/file2-`date --iso-8601=second`
R: Files:
file2-2019-03-08T11:36:30-06:00
file2-2019-03-08T11:36:35-06:00
```

> Why is it so?

[CFEngine expands variables wrapped in `$()`{.verbatim} and
`${}`{.verbatim}](https://docs.cfengine.com/docs/3.13/reference-language-concepts-variables.html#scalar-referencing-and-expansion)
and [functions are
skipped](https://docs.cfengine.com/docs/3.12/reference-functions.html#function-skipping)
(e.g.
[execresult()](https://docs.cfengine.com/docs/3.12/reference-functions-execresult.html)
)if they have parameters that are variables that do not dereference.

### [DONE]{.done .DONE} Scheduled for \[2019-05-13 Mon\] {#scheduled-for-2019-05-13-mon}
