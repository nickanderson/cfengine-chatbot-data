``` {.cfengine3 tangle="finding_processes_that_are_running.cf"}
bundle agent __main__
{
  reports:
      "findprocesses() returns:$(const.n)$(with)"
        with => string_mustache( '{{%-top-}}', findprocesses( "emacs" ));

}
```
