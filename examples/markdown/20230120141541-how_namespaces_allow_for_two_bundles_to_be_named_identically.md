``` {.cfengine3 include-stdlib="t" log-level="info" exports="both" tangle="how_namespaces_allow_for_two_bundles_to_be_named_identically.cf"}
bundle agent __main__
{
   methods:
    "" usebundle => my_bundle("first");
    "" usebundle => default:my_bundle("second");
    "" usebundle => nickanderson:my_bundle("third");
}
bundle agent my_bundle(input)
{
  reports:
    "Hello from $(this.namespace):$(this.bundle). I got $(input)";
}
body file control
{
    namespace => "nickanderson";
}
bundle agent my_bundle(input)
{
  reports:
    "Goodbye from $(this.namespace):$(this.bundle). I got $(input)";
}
```

``` example
R: Hello from default:my_bundle. I got first
R: Hello from default:my_bundle. I got second
R: Goodbye from nickanderson:my_bundle. I got third
```
