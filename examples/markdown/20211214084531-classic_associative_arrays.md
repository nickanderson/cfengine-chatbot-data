:::: captioned-content
::: caption
Example Policy
:::

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both" extra-opts="--show-evaluated-vars=default:main\\\\."}
bundle agent __main__
{
  vars:
      "foo" string => "Bar";
      "foo" string => "Bar";
      "foo[1]" string => "idx1";
      "foo[2]" string => "idx2";
      "i" slist => getindices( foo );

  reports:
    "Indices of foo: $(i)";
    "foo[$(i)] == $(foo[$(i)])";
    "foo: $(foo)";
}
```
::::

``` example
R: Indices of foo: 1
R: Indices of foo: 2
R: foo[1] == idx1
R: foo[2] == idx2
R: foo: Bar
Variable name                            Variable value                                               Meta tags                                Comment                                 
default:main.foo                         Bar                                                          source=promise                                                                   
default:main.foo[1]                      idx1                                                         source=promise                                                                   
default:main.foo[2]                      idx2                                                         source=promise                                                                   
default:main.i                            {"1","2"}                                                   source=promise                                                                   
```

What if foo is an slist?

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both" extra-opts="--show-evaluated-vars=default:main\\\\."}
bundle agent __main__
{
  vars:
      "foo" slist => { "bof", "of", "deez", "nuts" };
      "foo[1]" string => "idx1";
      "foo[2]" string => "idx2";
      "i" slist => getindices( foo );

  reports:
    "Indices of foo: $(i)";
    "foo[$(i)] == $(foo[$(i)])";
    "foo: $(foo)";
}
```

``` example
R: Indices of foo: 1
R: Indices of foo: 2
R: foo[1] == idx1
R: foo[2] == idx2
R: foo: bof
R: foo: of
R: foo: deez
R: foo: nuts
Variable name                            Variable value                                               Meta tags                                Comment                                 
default:main.foo                          {"bof","of","deez","nuts"}                                  source=promise                                                                   
default:main.foo[1]                      idx1                                                         source=promise                                                                   
default:main.foo[2]                      idx2                                                         source=promise                                                                   
default:main.i                            {"1","2"}                                                   source=promise                                                                   
```

This is incompatible withour ability to munch a classic associiatve
array into a data container

it took the slist where foo was explitly exactly defined, didnt consider
the associative collection

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both" extra-opts="--show-evaluated-vars=default:main\\\\."}
bundle agent __main__
{
  vars:
      "foo" slist => { "bof", "of", "deez", "nuts" };
      "foo[1]" string => "idx1";
      "foo[2]" string => "idx2";
      "i" slist => getindices( foo );
      "d" data => mergedata( foo );

  reports:
    "Indices of foo: $(i)";
    "foo[$(i)] == $(foo[$(i)])";
    "foo: $(foo)";
}
```

``` example
R: Indices of foo: 1
R: Indices of foo: 2
R: foo[1] == idx1
R: foo[2] == idx2
R: foo: bof
R: foo: of
R: foo: deez
R: foo: nuts
Variable name                            Variable value                                               Meta tags                                Comment                                 
default:main.d                           ["bof","of","deez","nuts"]                                   source=promise                                                                   
default:main.foo                          {"bof","of","deez","nuts"}                                  source=promise                                                                   
default:main.foo[1]                      idx1                                                         source=promise                                                                   
default:main.foo[2]                      idx2                                                         source=promise                                                                   
default:main.i                            {"1","2"}                                                   source=promise                                                                   
```

------------------------------------------------------------------------

more shit

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both" extra-opts="--show-evaluated-vars=default:main\\\\."}
bundle agent __main__
{
  vars:
      "foo[1]" string => "idx1";
      "foo[2]" string => "idx2";
      "i" slist => getindices( foo );
      "d" data => mergedata( foo );

  reports:
    #  Print out the jsonified form:
    "foo: $(with)" with => storejson( mergedata( foo ) );
}
```

``` example
R: foo: {
  "1": "idx1",
  "2": "idx2"
}
Variable name                            Variable value                                               Meta tags                                Comment                                 
default:main.d                           {"1":"idx1","2":"idx2"}                                      source=promise                                                                   
default:main.foo[1]                      idx1                                                         source=promise                                                                   
default:main.foo[2]                      idx2                                                         source=promise                                                                   
default:main.i                            {"1","2"}                                                   source=promise                                                                   
```

# References

- [CFEngine Examples](id:38277465-771a-4db4-983a-8dfd434b1aff)
