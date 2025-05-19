``` {.cfengine3 include-stdlib="t" log-level="info" exports="both"}
body agent control
{
      abortbundleclasses => { "abort_bundle_.*"};
}

bundle agent __main__
{
  methods:
      "one";
      "two";
      "three";
}

bundle agent one
{
  classes:
      "abort_bundle_because_any" expression => "any";

  reports:
      "hello from $(this.bundle)";
}

bundle agent two
{
  reports:
      "hello from $(this.bundle)";
}

bundle agent three
{
  classes:
      "abort_bundle_because_i_exist"
        expression => fileexists( $(this.promise_filename) );

  reports:
      "hello from $(this.bundle)";
}
```

``` example
   error: Bundle 'one' aborted on defined class 'abort_bundle_because_any'
R: hello from two
   error: Bundle 'three' aborted on defined class 'abort_bundle_because_i_exist'
```
