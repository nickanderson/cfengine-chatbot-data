Being able to pass the whole data object (like how slists can be passed)
is the newer feature that applies only to data containers.

\> Does this mean that it passes as an argument a variable copy not a
pointer?

Yes, I believe so. The bundle called then has it\'s own full copy of the
data. A parameter can be re-defined within a bundle. This example shows
how the data container defined in main is passed into
`bundle agent one`{.verbatim} where it\'s defined as
`my_param`{.verbatim}. The original value is emitted, then
`my_param`{.verbatim} is re-defined within the bundle. Finally, after
returning to main, the reports promise shows that the original variable
`d`{.verbatim} still holds the same, original value.

:::: captioned-content
::: caption
Example Policy
:::

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both" tangle="datacontainers_passed_with_are_passed_as_a_copy_of_the_original_variable.cf"}
bundle agent __main__
{
  vars:
      "d" data => '{ "msg": "Hello, World!" }';
  methods:
      "" usebundle => one( @(d) );

  reports:
    "d is $(with)" with => storejson( @(d) );
}
bundle agent one(my_param)
{
  vars:
    report_param_reached::
      "my_param" data => '{"msg": "So long, farewell, auf Wiedersehen, adieu" }';

  reports:
    report_param_reached::
      "Redefined my_param as $(with)" with => storejson( my_param ), classes => results( "bundle", "report_param" );

    !report_param_reached::
      "Got my_param as $(with)" with => storejson( my_param ), classes => results( "bundle", "report_param" );

}
```
::::

``` example
R: Got my_param as {
  "msg": "Hello, World!"
}
R: Redefined my_param as {
  "msg": "So long, farewell, auf Wiedersehen, adieu"
}
R: d is {
  "msg": "Hello, World!"
}
```
