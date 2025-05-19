``` {.cfengine3 include-stdlib="t" log-level="info" exports="both" tangle="cfengine_automatically_canonifies_strings_used_to_define_classes.cf"}
bundle agent __main__
{
  vars:
      "my_str" string => "hello-world";

  classes:
      "$(my_str)";

  reports:
      "CFEngine automatically canonifies strings when defining classes";
      "You should canonify strings explicitly when testing against them.";

      "$(my_str) is a defined class!"
        comment => "Boy, that would be a surprise.",
        if => "$(my_str)";

      "$(with) from $(my_str) is a defined class!"
        comment => "Here we test against the canonified version of the string",
        if => "$(with)",
        with => canonify( "$(my_str)");
}
```

``` example
R: CFEngine automatically canonifies strings when defining classes
R: You should canonify strings explicitly when testing against them.
R: hello_world from hello-world is a defined class!
```
