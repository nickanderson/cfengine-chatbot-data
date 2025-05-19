**TLDR;** Both data containers and associative arrays can be passed by
name (`<namespace>:<bundle>.<variable>`{.verbatim}). Only data
containers can be passed directly as a full copy.

Associative arrays aka classic arrays are really a way of treating
individual variables (key value pairs) as a cohesive singular data
structure. They can usually be used interchangeably with a real data
type var. \"deprecated\" might be too strong of a word, at least for the
life of CFEngine 3, they will remain and there are things you can do
with them like dynamically constructing a data container with the
ability to influence individual key values that really isn\'t possible
to do natively with data vars. With data vars to do dynamic munging you
would end up calling out to some external tool.

> How would you pass the "associative-arrays" as argument to another
> bundle?

Since associative arrays are really not an individual thing, but rather
a way to treat a collection of individual things you can only pass then
by name. You can use something like `mergedata()` to convert an
associative array into a data container and then pass that.

> Should I use the data variable type instead?

I don\'t see any specific benefit of doing so.

> I know that you can use
> `<bundle_namespace>:<bundle_name>.<var_name>`{.verbatim}.

Yes. You can pass data vars that way also. If a bundle is designed
expecting a data structure to be passed by name then you could give the
same bundle either the name of a data type var or the name of an
associative array and it would behave nearly identically.

This example illustrates the **very similar** (but *not identical*)
behavior working with associative arrays vs data containers.

:::: captioned-content
::: caption
Example Policy
:::

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both" tangle="passing_classic_arrays_associative_arrays_vs_data_containers.cf"}
bundle agent __main__
{
  vars:
      "d_animals" data => '{
"dog": { "legs": 4, "tail": true, "names": [ "Fido", "Cooper", "Sandy" ] },
"dolphin": { "legs": 0, "tail": true, "names": [ "Flipper", "Duffy" ] }
      }';

      "arr_animals[cat][legs]" int => "4";
      "arr_animals[cat][tail]" string => "true";
      "arr_animals[cat][names]" slist => { "Fluffy", "Snowball", "Tabby" };
      "arr_animals[hampster][legs]" int => "4";
      "arr_animals[hampster][tail]" string => "true";
      "arr_animals[hampster][names]" slist => { "Skullcrusher", "Kimmy", "Fluffadoo" };


  methods:
      "Data vars can be passed directly as a full copy"
        usebundle => show( @(d_animals), "$(this.promiser)" );

      "Data vars can be passed by fully qualified name"
        usebundle => show( "$(this.namespace):$(this.bundle).d_animals", "$(this.promiser)" );

      # Classic Associatiave Arrays can not be passed as a direct copy, only as a fully qualified name
      # If you try to pass an associative array as a copy, you get an error because the referenced variable does not exist
      # error: List or container parameter 'default:main.arr_animals' not found while constructing scope 'show' - use @(scope.variable) in calling reference
      # "Trying to pass associaiteve array as a copy"
      #   usebundle => show( "@($(this.namespace):$(this.bundle).arr_animals)", "$(this.promiser)" );

      # Classic Associatiave Arrays can not be passed as a direct copy, only as a fully qualified name
      "Only pass associateive array by fully qualified name"
        usebundle => show( "$(this.namespace):$(this.bundle).arr_animals", "$(this.promiser)" );

      # You can even pass the name to access "part" of an associative array
      "You can pass a subtree of an associatevie array by name"
        usebundle => show( "$(this.namespace):$(this.bundle).arr_animals[cat]" , "$(this.promiser)" );

      # You can pass data containers by name too
      "You can pass a subtree of a data var by name"
        usebundle => show( "$(this.namespace):$(this.bundle).d_animals[dog]", "$(this.promiser)" );

}

bundle agent show( assoc_array_or_data, msg )
{
  reports:
      "$(msg)";
      "Got assoc_array_or_data as '$(with)'"
        with => type( assoc_array_or_data ),
        handle => "0_$(msg)";

      "$(with)"
        with => storejson( assoc_array_or_data ),
        if => strcmp( "data", type( assoc_array_or_data ) ),
        handle => "1_$(msg)";

      "$(with)"
        with => storejson( mergedata( $(assoc_array_or_data) ) ),
        if => strcmp( "string", type( assoc_array_or_data ) ),
        handle => "2_$(msg)";
}
```
::::

Output:

------------------------------------------------------------------------

``` example
R: Data vars can be passed directly as a full copy
R: Got assoc_array_or_data as 'data'
R: {
  "dog": {
    "legs": 4,
    "names": [
      "Fido",
      "Cooper",
      "Sandy"
    ],
    "tail": true
  },
  "dolphin": {
    "legs": 0,
    "names": [
      "Flipper",
      "Duffy"
    ],
    "tail": true
  }
}
R: Data vars can be passed by fully qualified name
R: Got assoc_array_or_data as 'string'
R: {
  "dog": {
    "legs": 4,
    "names": [
      "Fido",
      "Cooper",
      "Sandy"
    ],
    "tail": true
  },
  "dolphin": {
    "legs": 0,
    "names": [
      "Flipper",
      "Duffy"
    ],
    "tail": true
  }
}
R: Only pass associateive array by fully qualified name
R: Got assoc_array_or_data as 'string'
R: {
  "cat": {
    "legs": "4",
    "names": [
      "Fluffy",
      "Snowball",
      "Tabby"
    ],
    "tail": "true"
  },
  "hampster": {
    "legs": "4",
    "names": [
      "Skullcrusher",
      "Kimmy",
      "Fluffadoo"
    ],
    "tail": "true"
  }
}
R: You can pass a subtree of an associatevie array by name
R: Got assoc_array_or_data as 'string'
R: {
  "legs": "4",
  "names": [
    "Fluffy",
    "Snowball",
    "Tabby"
  ],
  "tail": "true"
}
R: You can pass a subtree of a data var by name
R: Got assoc_array_or_data as 'string'
R: {
  "legs": 4,
  "names": [
    "Fido",
    "Cooper",
    "Sandy"
  ],
  "tail": true
}
```
