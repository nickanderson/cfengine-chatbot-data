> How would you verify that `host002[vars][]`{.verbatim} has any values?

`getindices()` always returns a list (instead of not returning when
there is nothing to return). So, you can check it\'s *length*.

:::: captioned-content
::: caption
Example Policy
:::

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both" extra-opts="--show-evaluated-vars=default:main\\\\." tangle="check_if_there_are_any_subkeys_under_a_classic_array_key.cf"}
bundle agent __main__
{
  vars:
      # "host002[vars][nvs_test02_config_enabled]" string => "1";
      "host002_idx" slist => getindices("host002[vars]");
      "l" int => length( host002_idx );

  reports:
      "$(this.bundle):exists"
        if => isvariable("host002_idx");

      "There are no subkeys under host002[vars] with strcmp()"
        if => strcmp( length( host002_idx ), 0 );

      "There are no subkeys under host002[vars] with eval()"
        if => eval( "$(l) == 0", class, infix );

      "There are no subkeys under host002[vars] with some()"
        unless => some( ".*", length( host002_idx ) );
}
```
::::

``` example
R: main:exists
Variable name                            Variable value                                               Meta tags                                Comment
default:main.host002_idx                                                                              source=promise
default:main.l                           0                                                            source=promise
```
