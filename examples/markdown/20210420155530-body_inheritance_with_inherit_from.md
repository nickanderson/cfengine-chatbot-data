```{=org}
#+roam_tags: CFEngine-example
```
> lets say i have this classes body:

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both"}
body classes set_some_fancy_classes ( x, y) 
{
        promise_kept => { "some_fancy_class_${x}_${y}_kept" }; 
        promise_repaired => { "some_fancy_class_${x}_${y}_repaired" };  
        repair_failed => { "some_fancy_class_${x}_${y}_notkept" }; 
        repair_denied => { "some_fancy_class_${x}_${y}_notkept" }; 
        repair_timeout => { "some_fancy_class_${x}_${y}_notkept" }; 
} 
```

> which is used like this:

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both"}
commands: 
   "/bin/true" 
       handle => "some meaningful string",
       classes => set_some_fancy_classes( "$(this.promiser)", 
"$(this.handle)" );
```

> so far so good, the one thing that i dont get is, how do i set
> \"additional\" classes in this scenario, for example i want to set
> another class like this (not handled by the mentioned classes body):

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both"}

commands: 
    "/bin/true" 
        handle => "some meaningful string",
        classes => set_some_fancy_classes( "$(this.promiser)", 
"$(this.handle)" ),
        classes = if_repaired("some_class_to_handle_dependencies");
```

> problem is, \"last one wins\" rule is firing, so no luck here.
>
> i read about the \"inherit~from~\" attribute, but i have no clue how
> to implement this or if it is possible anyway. the documentation /
> examples left me clueless.
>
> i want to set the classes from the \" set~somefancyclasses~\" body
> PLUS the class from the \"if~repaired~\" attribute.

Easily record content

Hi Chris,

Right, you should not attach multiple of the same attribute to the same
promise. Perhaps we should make the parser disallow it all together
because it\'s not an expected use.

I believe this example implements what you are looking for.

We define a new classes body
`my_set_some_extra_fancy_classes`{.verbatim}, it takes 3 arguments where
the third argument is an additional class to set when the promise is
*repaired*. Note, since we are setting `promise_repaired`{.verbatim}
differently in `my_set_some_extra_fancy_classes`{.verbatim} than
`my_set_some_fancy_classes`{.verbatim} we have to completely re-define
it.

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both" extra-opts="--show-evaluated-classes=some_" tangle="example.cf" command-in-result="t"}
bundle agent __main__
{
  commands:
      "/bin/true"
        handle => "some meaningful string",
        classes => my_set_some_extra_fancy_classes( "$(this.promiser)",
                                              "$(this.handle)",
                                              "some_class_to_handle_dependencies" );

      "/bin/false"
        handle => "some meaningless string",
        classes => my_set_some_extra_fancy_classes( "$(this.promiser)",
                                              "$(this.handle)",
                                              "some_class_to_handle_dependencies" );
}

body classes my_set_some_extra_fancy_classes( x, y, z)
# @breif In addition to the classes set by `set_some_fancy_classes` define `z` when the promise is repaired 
{
        inherit_from => set_some_fancy_classes( $(x), $(y) );
        promise_repaired => { "some_fancy_class_${x}_${y}_repaired", $(z) };
}

body classes set_some_fancy_classes ( x, y)
{
        promise_kept => { "some_fancy_class_${x}_${y}_kept" };
        promise_repaired => { "some_fancy_class_${x}_${y}_repaired" };
        repair_failed => { "some_fancy_class_${x}_${y}_notkept" };
        repair_denied => { "some_fancy_class_${x}_${y}_notkept" };
        repair_timeout => { "some_fancy_class_${x}_${y}_notkept" };
}
```

``` example
# cf-agent --no-lock --log-level info --show-evaluated-classes=some_ --file example.cf
    info: Executing 'no timeout' ... '/bin/true'
    info: Completed execution of '/bin/true'
    info: Executing 'no timeout' ... '/bin/false'
   error: Finished command related to promiser '/bin/false' -- an error occurred, returned 1
    info: Completed execution of '/bin/false'
Class name                                                   Meta tags                               
some_class_to_handle_dependencies                                                                    
some_fancy_class__bin_false_some_meaningless_string_notkept                                          
some_fancy_class__bin_true_some_meaningful_string_repaired                                           
```

Here I use `--show-evaluated-classes`{.verbatim} which lists the
*namespace* scoped classes defined at the end of the agent run filtered
for those starting with `some_`{.verbatim}. We can see that both
`some_class_to_handle_dependencies`{.verbatim} and
`some_fancy_class__bin_true_some_meaningful_string_kept`{.verbatim} get
defined.

```{=org}
#+RESULTS:
```
``` example
# cf-agent --no-lock --log-level info --show-evaluated-classes=some_ --file example.cf
    info: Executing 'no timeout' ... '/bin/true'
    info: Completed execution of '/bin/true'
Class name                                                   Meta tags                               
some_class_to_handle_dependencies                                                                    
some_fancy_class__bin_true_some_meaningful_string_kept                                               
```

``` {.cfengine3 tangle="body_inheritance_with_inherit_from.cf"}
accessedbefore
bundle agent __main__
{


}
```
