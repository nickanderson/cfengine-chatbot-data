``` {.cfengine3 tangle="extracting_an_array_from_a_deep_json_key.cf"}
bundle agent __main__
{
  %?
}
```

``` {.cfengine3 exports="both" wrap="EXAMPLE"}
bundle agent main
{
  vars:
    "foodata"
      data => '{"foo": {"bar": ["abc", "xyz"]} }';
    "foostring" string => join(", ", @(foodata[foo][bar]));

  reports:
    "CFEngine $(sys.cf_version)";
    "foostring = $(foostring)";
}
```

``` example
R: CFEngine 3.11.0a.334d1fc
R: foostring = abc, xyz
```

You can also extract the list:

``` {.cfengine3 exports="both" wrap="EXAMPLE"}
bundle agent main
{
  vars:
    "foodata"
      data => '{"foo": {"bar": ["abc", "xyz"]} }';

    # Pick it into its own data container
    "foodatalist"
      data => mergedata( "foodata[foo][bar]" );

    # OR pull out a list

    "foolist"
      slist => getvalues("foodata[foo][bar]");

  reports:
    "CFEngine $(sys.cf_version)";
    "foodatalist = $(foodatalist)";
    "foodlist = $(foolist)";
}
```

``` example
R: CFEngine 3.11.0a.334d1fc
R: foodatalist = abc
R: foodatalist = xyz
R: foodlist = abc
R: foodlist = xyz
```
