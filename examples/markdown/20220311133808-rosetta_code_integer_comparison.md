<https://rosettacode.org/wiki/Integer_comparison>

Get two integers from the user.

Then, display a message if the first integer is:

less than, equal to, or greater than

the second integer.

Test the condition for each case separately, so that all three
comparison operators are used in the code.

:::: captioned-content
::: caption
Example Policy
:::

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both" tangle="rosetta_code_integer_comparison.cf"}
bundle agent __main__
{
  vars:
      "first_integer" int => "10";
      "second_integer" int => "9";

  reports:
     "The first integer ($(first_integer)) is less than the second integer ($(second_integer))."
        if => islessthan( "$(first_integer)", "$(second_integer)" );
     "The first integer ($(first_integer)) is equal to the second integer ($(second_integer))."
        if => eval( "$(first_integer)==$(second_integer)", "class", "infix" );
     "The first integer ($(first_integer)) is greater than the second integer ($(second_integer))."
        if => isgreaterthan( "$(first_integer)", "$(second_integer)" );

}
```
::::

``` example
R: The first integer (10) is greater than the second integer (9).
```
