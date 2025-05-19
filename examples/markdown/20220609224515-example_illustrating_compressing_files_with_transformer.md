:::: captioned-content
::: caption
Example Policy
:::

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both" tangle="example_illustrating_compressing_files_with_transformer.cf"}
bundle agent __main__
{
  files:
      "/tmp/example.txt"
        content => "Hello World";

      "/tmp/example.txt"
        transformer => "/usr/bin/gzip $(this.promiser)";

  reports:
      "$(with)" with => join( ",", lsdir( "/tmp", "example.*", "true" ));
}
```
::::

``` example
    info: Updated content of '/tmp/example.txt' with content 'Hello World'
    info: Transforming '/tmp/example.txt' with '/usr/bin/gzip /tmp/example.txt'
    info: Transformed '/tmp/example.txt' with '/usr/bin/gzip /tmp/example.txt' 
    info: Transformer '/tmp/example.txt' => '/usr/bin/gzip /tmp/example.txt' seemed to work ok
R: /tmp/example.txt.gz
```

# When the promiser doesn\'t exist

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both"}
bundle agent __main__
{
  files:
      "/tmp/an-absent-file.txt"
        delete => tidy;

      "/tmp/an-absent-file.txt"
        transformer => "/usr/bin/gzip $(this.promiser)";

  reports:
      "$(with)" with => join( ",", lsdir( "/tmp", "an-absent.*", "true" ));
}
```
