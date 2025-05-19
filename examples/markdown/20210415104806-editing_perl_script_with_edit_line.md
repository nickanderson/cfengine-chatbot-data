```{=org}
#+roam_tags: CFEngine-example
```
<https://gist.github.com/nickanderson/ea2428376b51758655e90e455d65451b>

:::: captioned-content
::: caption
Starting file
:::

``` {.txt tangle="/tmp/openldap-migration.txt"}
I've got a lovely bunch of coconuts 
$DEFAULT_BASE = "dc=padl,dc=com";
1;
```
::::

:::: captioned-content
::: caption
Policy to edit DEFAULT~BASE~
:::

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both" tangle="editing_perl_script_with_edit_line.cf"}
bundle agent __main__
{
  methods:
      "seed";
      "water";
      "harvest";
}
bundle agent seed
{
  vars:
      "my_file" string => "/tmp/openldap-migration.txt";

  reports:
      "Before Policy: $(my_file)" printfile => cat( "$(my_file)" );
}

bundle agent water
{
  files:
      "$(seed.my_file)"
        edit_line => dollar_prefixed_key_space_equal_doublequoted_value( "DEFAULT_BASE", "dc=dynafabric,dc=local" );

}

bundle agent harvest
{
  reports:
      "After Policy: $(seed.my_file)" printfile => cat( "$(seed.my_file)" );
}
bundle edit_line dollar_prefixed_key_space_equal_doublequoted_value( KEY, VALUE )
{
  replace_patterns:

      `\Q$\E$(KEY)\s+=\s+(?!"$(VALUE)").*`
        replace_with => value( `$(const.dollar)$(KEY) = "$(VALUE)";` );
}
```
::::

``` example
R: Before Policy: /tmp/openldap-migration.txt
R: I've got a lovely bunch of coconuts 
R: $DEFAULT_BASE = "dc=padl,dc=com";
R: 1;
    info: Replaced pattern '\Q$\EDEFAULT_BASE\s+=\s+(?!"dc=dynafabric,dc=local").*' in '/tmp/openldap-migration.txt'
    info: replace_patterns promise '\Q$\EDEFAULT_BASE\s+=\s+(?!"dc=dynafabric,dc=local").*' repaired
    info: Edited file '/tmp/openldap-migration.txt'
    info: files promise '/tmp/openldap-migration.txt' repaired
R: After Policy: /tmp/openldap-migration.txt
R: I've got a lovely bunch of coconuts 
R: $DEFAULT_BASE = "dc=dynafabric,dc=local";
R: 1;
```
