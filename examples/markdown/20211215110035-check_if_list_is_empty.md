```{=org}
#+filetags: :CFEngine:example:
```
:::: captioned-content
::: caption
Check if an slist is empty
:::

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both"}
bundle agent __main__
{
      vars:
      "my_empty_list" slist => { };

  reports:
      'my_empty_list is empty if => none( ".*", my_empty_list )'
        if => none( ".*", my_empty_list );

       'my_empty_list is empty if => isgreaterthan( 0, len( my_empty_list ) )'
        if => islessthan( length( my_empty_list ), 1 );

       'my_empty_list is empty if => eval( "$(with) == 0", "class", "infix" )'
         with => length( my_empty_list ),
         if => eval( "$(with) == 0", "class", "infix" );
}
```
::::

``` example
R: my_empty_list is empty if => none( ".*", my_empty_list )
R: my_empty_list is empty if => isgreaterthan( 0, len( my_empty_list ) )
R: my_empty_list is empty if => eval( "0 != 0", "class", "infix" )
```

**References:**

- [CFEngine Examples](id:38277465-771a-4db4-983a-8dfd434b1aff)
- [none()](id:3ed3e893-6b36-478a-b284-9538ecd7464f)
- [islessthan()](id:20975dc3-bf18-4b67-a0f9-a0717f044102)
- [length()](id:22b0b944-3335-40c8-957c-0e6a474d1c85)
- [eval()](id:24647e3a-2af2-4460-897d-5b539bff2171)
