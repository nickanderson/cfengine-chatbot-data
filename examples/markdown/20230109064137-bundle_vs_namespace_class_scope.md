Classes defined as the result of a classes type promise in
`common`{.verbatim} bundles have a default scope of \"namespace\".
Classes defined as the result of a classes type promise in
`agent`{.verbatim} bundles have a default scope of \"bundle\".

This default can be overridden with the `scope`{.verbatim} attribute.

For example:

``` cfengine3
bundle common def
{
  classes:
      "def_namespace_scoped_aka_global_class"
        expression => "any";

      "def_bundle_scoped_aka_local_class"
        expression => "any", scope => "bundle";

}
bundle agent main
{
  classes:
      "main_bundle_scoped_aka_local_class"
        expression => "any";

      "main_namespace_scoped_aka_global_class"
        expression => "any",
        scope => "namespace";
}
```

Classes defined as the result of a promise outcome defined by a
`classes body`{.verbatim} have a default scope of
**`namespace`{.verbatim}**. You can override this using the
[`scope`{.verbatim} `classes body`{.verbatim}
attribute](https://docs.cfengine.com/lts/reference-promise-types.html#scope).

For example:

``` cfengine3
bundle agent main
{
  commands:
      "/bin/true"
        classes => if_repaired("my_class"),
        comment => "Different classes bodies use different scopes. Some allow
                   you to pass the scope in as a parameter. Try the `results`
                   body.";

      "/bin/false"
        classes => results("bundle", "my_bundle_scope_class_with_false"),
        handle => "failed_promise_with_results_body";


      "/bin/true"
        classes => results("bundle", "my_bundle_scope_class_with_true"),
        handle => "repaired_promise_with_results_body";

  vars:
    "classes" slist => classesmatching("my_.*");

  reports:
    "Class visible in $(this.bundle): '$(classes)'"
      handle => "main_classes_report";

  methods:
    "another_bundle"
      depends_on => { "main_classes_report" };

 }

bundle agent another_bundle
{
  vars:
      "classes" slist => classesmatching("my_.*");

  reports:
      "Class visible in $(this.bundle): '$(classes)'"
      handle => "another_bundle_classes_report";

}
```
