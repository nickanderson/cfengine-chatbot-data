``` {.cfengine3 tangle="using_regular_expressions_in_files_promisers.cf" info="yes" include-stdlib="t" verbose-mode="nil" exports="both"}
bundle agent example
{

  files:
    "/tmp/example-1" create => "true";
    "/tmp/example-2" create => "true";

    "/tmp/example-.*"
      delete => tidy;

  reports:
      "CFEngine $(sys.cf_version)";

}
bundle agent __main__
{
methods:
      "example";
}
```

``` example
    info: Created file '/tmp/example-1', mode 0600
    info: Created file '/tmp/example-2', mode 0600
    info: Deleted file '/tmp/example-2'
    info: Deleted file '/tmp/example-1'
R: CFEngine 3.21.0
```
