```{=org}
#+roam_tags: CFEngine-example bundle_return_value_index useresult methods usebundle
```
:::: captioned-content
::: caption
Example illustrating how a bundle can return data to a calling bundle
:::

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both"}
bundle agent two
{
}
bundle agent one
{

  reports:
      "hi" bundle_return_value_index => "greeting";
      "byeeee" bundle_return_value_index => "farewell";
}

bundle agent __main__
{
  methods:
      "one"
        useresult => "one_returned"; # Name of the variable where reports
                                     # returned by the bundle should be stored.
                                     # - This is a /classic/ array
                                     # - Checkout it's /keys/ and /values/;


  reports:
      "$(with)" with => string_mustache( "{{%-top-}}", mergedata( "one_returned" ));
      "Keys of 'one_returned': $(with)" with => join( ", ", getindices( "one_returned" ));
      "one_returned[greeting] == '$(one_returned[greeting])'";
      "one_returned[farewell] == '$(one_returned[farewell])'";
}
```
::::

``` example
R: {
  "farewell": "byeeee",
  "greeting": "hi"
}
R: Keys of 'one_returned': greeting, farewell
R: one_returned[greeting] == 'hi'
R: one_returned[farewell] == 'byeeee'
```

Beware promise locking ....

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both"}
bundle agent echo( param1, param2 )
{
  commands:

      "/bin/echo $(param1)"; # What would you expect to happen here, consider
                             # mutliple runs wher some params have the same
                             # value across each different run.
      "/bin/echo $(param2)";

  reports:
      "hi" bundle_return_value_index => "greeting";
      "byeeee" bundle_return_value_index => "farewell";
      "$(param1)" bundle_return_value_index => "what_will_i_hold";
      "$(param1)" bundle_return_value_index => "param1";
      "$(param2)" bundle_return_value_index => "param2";
}

bundle agent __main__
{
  methods:
      "First call"
        usebundle => one( "param1", "param2" ),
        useresult => "one_returned"; # Name of the variable where reports
                                     # returned by the bundle should be stored.
                                     # - This is a /classic/ array
                                     # - Checkout it's /keys/ and /values/;

      "Second call"
        usebundle => one( "param2", "param1" ),
        useresult => "one_returned"; # Name of the variable where reports
                                     # returned by the bundle should be stored.
                                     # - This is a /classic/ array
                                     # - Checkout it's /keys/ and /values/;


  reports:
      "$(with)" with => string_mustache( "{{%-top-}}", mergedata( "one_returned" ));
      "Keys of 'one_returned': $(with)" with => join( ", ", getindices( "one_returned" ));
      "one_returned[greeting] == '$(one_returned[greeting])'";
      "one_returned[farewell] == '$(one_returned[farewell])'";
}
```
