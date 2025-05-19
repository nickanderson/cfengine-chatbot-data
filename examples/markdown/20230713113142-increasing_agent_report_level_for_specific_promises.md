`action`{.verbatim} bodies can specify `log_level`{.verbatim}
(<https://docs.cfengine.com/docs/3.21/reference-promise-types.html#log_level>)
and `report_level`{.verbatim}
(<https://docs.cfengine.com/docs/3.21/reference-promise-types.html#report_level>).
`log_level`{.verbatim} influences what is sent to syslog and
`report_level`{.verbatim} influences standard output. These are useful
for **increasing**, not decreasing the level of output from the global
default. So, if you ran `cf-agent` with no log level specified you could
for a single promise increase it\'s level to inform for example, but if
you run `cf-agent --log-level inform` you could not reduce an individual
promise log level to `error`{.verbatim}.

Here is a small example to illustrate the behavior:

:::: captioned-content
::: caption
Example Policy
:::

``` {#example .cfengine3 include-stdlib="t" exports="both" tangle="./increasing_agent_report_level_for_specific_promises.cf" command-in-result="t"}
bundle agent __main__
{
    methods:
      "init";
      "example";
}
bundle agent init
{
  files:
    "/tmp/[1-3].txt"
      delete => default:tidy,
      action => default:report_level_x( "error" );
}
bundle agent example
{
  files:
    "/tmp/1.txt"
      content => "default report level";

    "/tmp/2.txt"
      content => "promise specific inform report level",
      action => default:report_level_x( "inform" );

    "/tmp/3.txt"
      content => "promise specific verbose report level",
      action => default:report_level_x( "verbose" );


  reports:
    "/tmp/1.txt" printfile => default:cat( "$(this.promiser)" );
    "/tmp/2.txt" printfile => default:cat( "$(this.promiser)" );
    "/tmp/3.txt" printfile => default:cat( "$(this.promiser)" );

}
body action report_level_x( x )
{
    report_level => "$(x)";
}
```
::::

``` example
# cf-agent --no-lock --file ./increasing_agent_report_level_for_specific_promises.cf
    info: Created file '/tmp/2.txt', mode 0600
    info: Updated file '/tmp/2.txt' with content 'promise specific inform report level'
 verbose: P: .........................................................
 verbose: P: BEGIN promise 'promise_cfengine3_SlMu4Z_25' of type "files" (pass 1)
 verbose: P:    Promiser/affected object: '/tmp/3.txt'
 verbose: P:    Part of bundle: example
 verbose: P:    Base context class: any
 verbose: P:    Stack path: /default/main/methods/'example'/default/example/files/'/tmp/3.txt'[1]
 verbose: Using literal pathtype for '/tmp/3.txt'
 verbose: No mode was set, choose plain file default 0600
 verbose: Additional promise info: source path '/home/nickanderson/org/roam/CFEngine/examples/cfengine3-SlMu4Z' at line 25
    info: Created file '/tmp/3.txt', mode 0600
 verbose: Replacing '/tmp/3.txt' with content 'promise specific verbose report level'
 verbose: Additional promise info: source path '/home/nickanderson/org/roam/CFEngine/examples/cfengine3-SlMu4Z' at line 25
    info: Updated file '/tmp/3.txt' with content 'promise specific verbose report level'
 verbose: Additional promise info: source path '/home/nickanderson/org/roam/CFEngine/examples/cfengine3-SlMu4Z' at line 25
 verbose: files promise '/tmp/3.txt' repaired
 verbose: A: Promise REPAIRED
 verbose: P: END files promise (/tmp/3.txt)
R: /tmp/1.txt
R: default report level
R: /tmp/2.txt
R: promise specific inform report level
R: /tmp/3.txt
R: promise specific verbose report level
 verbose: P: .........................................................
 verbose: P: BEGIN promise 'promise_cfengine3_SlMu4Z_25' of type "files" (pass 2)
 verbose: P:    Promiser/affected object: '/tmp/3.txt'
 verbose: P:    Part of bundle: example
 verbose: P:    Base context class: any
 verbose: P:    Stack path: /default/main/methods/'example'/default/example/files/'/tmp/3.txt'[1]
 verbose: Using literal pathtype for '/tmp/3.txt'
 verbose: P: .........................................................
 verbose: P: BEGIN promise 'promise_cfengine3_SlMu4Z_25' of type "files" (pass 3)
 verbose: P:    Promiser/affected object: '/tmp/3.txt'
 verbose: P:    Part of bundle: example
 verbose: P:    Base context class: any
 verbose: P:    Stack path: /default/main/methods/'example'/default/example/files/'/tmp/3.txt'[1]
 verbose: Using literal pathtype for '/tmp/3.txt'
```

Then, when run again with inform as the default, we can see that the
file deletion still comes out as info, it\'s not suppressed by setting
it\'s `report_level`{.verbatim} to `error`{.verbatim} because this
attribute can only **increase** the verbosity.

```{=org}
#+call: example() :log-level inform
```
``` example
# cf-agent --no-lock --log-level inform --file ./increasing_agent_report_level_for_specific_promises.cf
    info: Deleted file '/tmp/1.txt'
    info: Deleted file '/tmp/2.txt'
    info: Deleted file '/tmp/3.txt'
    info: Created file '/tmp/1.txt', mode 0600
    info: Updated file '/tmp/1.txt' with content 'default report level'
    info: Created file '/tmp/2.txt', mode 0600
    info: Updated file '/tmp/2.txt' with content 'promise specific inform report level'
 verbose: P: .........................................................
 verbose: P: BEGIN promise 'promise_cfengine3_MkYeF1_25' of type "files" (pass 1)
 verbose: P:    Promiser/affected object: '/tmp/3.txt'
 verbose: P:    Part of bundle: example
 verbose: P:    Base context class: any
 verbose: P:    Stack path: /default/main/methods/'example'/default/example/files/'/tmp/3.txt'[1]
 verbose: Using literal pathtype for '/tmp/3.txt'
 verbose: No mode was set, choose plain file default 0600
 verbose: Additional promise info: source path '/home/nickanderson/org/roam/CFEngine/examples/cfengine3-MkYeF1' at line 25
    info: Created file '/tmp/3.txt', mode 0600
 verbose: Replacing '/tmp/3.txt' with content 'promise specific verbose report level'
 verbose: Additional promise info: source path '/home/nickanderson/org/roam/CFEngine/examples/cfengine3-MkYeF1' at line 25
    info: Updated file '/tmp/3.txt' with content 'promise specific verbose report level'
 verbose: Additional promise info: source path '/home/nickanderson/org/roam/CFEngine/examples/cfengine3-MkYeF1' at line 25
 verbose: files promise '/tmp/3.txt' repaired
 verbose: A: Promise REPAIRED
 verbose: P: END files promise (/tmp/3.txt)
R: /tmp/1.txt
R: default report level
R: /tmp/2.txt
R: promise specific inform report level
R: /tmp/3.txt
R: promise specific verbose report level
 verbose: P: .........................................................
 verbose: P: BEGIN promise 'promise_cfengine3_MkYeF1_25' of type "files" (pass 2)
 verbose: P:    Promiser/affected object: '/tmp/3.txt'
 verbose: P:    Part of bundle: example
 verbose: P:    Base context class: any
 verbose: P:    Stack path: /default/main/methods/'example'/default/example/files/'/tmp/3.txt'[1]
 verbose: Using literal pathtype for '/tmp/3.txt'
 verbose: P: .........................................................
 verbose: P: BEGIN promise 'promise_cfengine3_MkYeF1_25' of type "files" (pass 3)
 verbose: P:    Promiser/affected object: '/tmp/3.txt'
 verbose: P:    Part of bundle: example
 verbose: P:    Base context class: any
 verbose: P:    Stack path: /default/main/methods/'example'/default/example/files/'/tmp/3.txt'[1]
 verbose: Using literal pathtype for '/tmp/3.txt'
```
