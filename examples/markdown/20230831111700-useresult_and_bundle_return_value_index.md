:::: captioned-content
::: caption
Example Policy
:::

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both" tangle="useresult_and_bundle_return_value_index.cf"}
bundle agent __main__
{
  methods:

      "Call a bundle, define an associative array as a byproduct"
        usebundle => example_bundle_return_value_index_and_useresult("value 1"),
        useresult => "my_array";

      "Call a bundle, define an associative array as a byproduct"
        usebundle => example_bundle_return_value_index_and_useresult(""),
        useresult => "my_array";

  reports:
      "JSON representation of 'my_array' :$(const.n) $(with)"
        with => storejson( mergedata( my_array ) );
}
bundle agent example_bundle_return_value_index_and_useresult(param)
{
  reports:
      "Value 0"
        bundle_return_value_index => "0";

      "Value 2"
        bundle_return_value_index => "two";

      "$(with)"
        with => string_upcase( "$(param)" ), # Here we upcase the value of param and use it as the value for the key returned
        bundle_return_value_index => "$(param)"; # Here the parameter value is used to set the key that is returned

}
```
::::

``` example
R: JSON representation of 'my_array' :
 {
  "0": "Value 0",
  "two": "Value 2",
  "value 1": "VALUE 1"
}
```
