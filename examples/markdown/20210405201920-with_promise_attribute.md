```{=org}
#+roam_tags: CFEngine
```
``` {.cfengine3 tangle="with_promise_attribute.cf"}
bundle agent test
{
    vars:
      "hello" string => "world";
      "colors" slist => { "red", "blue", "green" };
}
bundle agent __main__
{
  reports:
    "$(with)" with => "=with= takes a /string/";
    "$(with)" with => concat( "It can use functions ",
                              "that return strings" );
    "$(with)" with => storejson( bundlestate( "test" ) );
}
```
