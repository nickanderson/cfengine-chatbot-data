```{=org}
#+roam_tags: CFEngine-example
```
The policy free and overridable are synonyms. The policy constant is
deprecated, and has no effect. All variables are free or overridable by
default which means the variables values may be changed.

The policy ifdefined applies only to lists and implies that unexpanded
or undefined lists are dropped. The default behavior is otherwise to
retain this value as an indicator of the failure to quench the variable
reference, for example:

:::: captioned-content
::: caption
Extending a list by including itself
:::

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both"}
bundle agent __main__
{
  vars:
      "l"
        slist => { "a", "b", "c", @(missing), "d", "e", "f" },
        policy => "ifdefined"; # @(missing) should be dropoped

      "n"
        slist => { "a", "b", "c", @(missing), "d", "e", "f" }; 
        # @(missing) be kept as an indicator of the failure to quench the variable reference
        # BUG/Mismatch with docs
        # This seems to be /skipped/ because of the use of an undefined variable


  reports:
      "vars: l    == $(with)" with => join( ", ", @(l) );
      "vars: n    == $(with)" with => join( ", ", @(n) );
}
```
::::

``` example
R: vars: l    == a, b, c, d, e, f
R: vars: n    == $(with)
```
