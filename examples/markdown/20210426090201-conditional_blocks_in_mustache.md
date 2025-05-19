```{=org}
#+roam_tags: CFEngine-example
```
[CFEngine Examples](id:38277465-771a-4db4-983a-8dfd434b1aff)

This small example illustrates the mustache syntax, showing that
`^`{.verbatim} is a negative match and `#`{.verbatim} is a positive
match.

``` {.cfengine3 tangle="conditional_blocks_in_mustache.cf"}
bundle agent __main__
{
    reports:
      "$(with)" with => string_mustache( concat( "CFEngine $(sys.cf_version)$(const.n)",
                                                 "{{^classes.this_class_is_not_defined}}",
                                                   "The class 'this_class_is_not_defined' is indeed, not defined",
                                                 "{{/classes.this_class_is_not_defined}}$(const.n)",
                                                 "{{#classes.cfengine}}",
                                                   "The class 'cfengine' is defined.",
                                                 "{{/classes.cfengine}}" ),
                                         datastate() );
}
```
