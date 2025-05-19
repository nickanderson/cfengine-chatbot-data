Note that only *namespace* scoped classes will be emitted.

:::: captioned-content
::: caption
/tmp/CFE-3887.cf
:::

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both" extra-opts="--show-evaluated-classes=MyClass" command-in-result="t" tangle="/tmp/CFE-3887.cf"}
bundle agent __main__
{
  classes:
      "MyClass" scope => "namespace";
      "MyClass2" scope => "bundle";

  methods:
      "MyNamespace:MyNamespacedBundle";
}

body file control
{
      namespace => "MyNamespace";
}

bundle agent MyNamespacedBundle
{
  classes:
      "MyClass" scope => "namespace";
      "MyClass2" scope => "bundle";
}
```
::::

``` example
# cf-agent --no-lock --log-level info --show-evaluated-classes=MyClass --file /tmp/CFE-3887.cf
Class name                                                   Meta tags                                Comment                                 
MyClass                                                      source=promise                                                                   
MyNamespace:MyClass                                          source=promise                                                                   
```

# References

- [CFEngine Examples](id:38277465-771a-4db4-983a-8dfd434b1aff)
