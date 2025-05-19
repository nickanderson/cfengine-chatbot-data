``` {.cfengine3 tangle="findfiles_up_aka_search_up.cf"}
bundle agent __main__
{
  classes:
      "have_dir"
        expression => isdir( "/tmp/cfengine/example/find/files/up/" );

  files:
      "/tmp/cfengine/example/find/files/up/." create => "true";
      "/tmp/cfengine/example/FINDTHIS" create => "true";
      "/tmp/cfengine/example/find/FINDTHAT" create => "true";

  reports:
    have_dir::
      "First file starting with FIND is $(with)"
        # with is processed before if/unless so we use a traditional class guard
        with => nth( findfiles_up( "/tmp/cfengine/example/find/files/up",
                                   "FIND*",
                                   inf), 0);
}
```

Seems bugged in 3.21.0 when you get multiple results

``` {.cfengine3 tangle="findfiles_up_aka_search_up.cf"}
bundle agent __main__
{
  files:
    "/tmp/cfengine/examples/find/files/up/." create => "true";
    "/tmp/cfengine/examples/FINDTHIS" create => "true";
    "/tmp/cfengine/examples/find/FINDTHAT" create => "true";

  reports:
    "$(with)"
      with => storejson( findfiles_up( "/tmp/cfengine/examples/find/files/up",
                                       "FIND*",
                                       inf)),
      if => isdir("/tmp/cfengine/examples/find/files/up");
}
```
