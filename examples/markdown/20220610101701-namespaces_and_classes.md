```{=org}
#+filetags: :CFEngine:example:
```
``` {.cfengine3 tangle="namespaces_and_classes.cf"}
bundle agent __main__
{
  methods:
      "MyNamespace:my_bundle";

  reports:

    a_bundle_scoped_class_in_my_namespaced_bundle::
      "In $(this.namespace):$(this.bundle) I see 'a_bundle_scoped_class_in_my_namespaced_bundle::'";

    !a_bundle_scoped_class_in_my_namespaced_bundle::
      "In $(this.namespace):$(this.bundle) I do not see 'a_bundle_scoped_class_in_my_namespaced_bundle::'";

    a_namespace_scoped_class_in_my_namespaced_bundle::
      "In $(this.namespace):$(this.bundle) I see 'a_namespace_scoped_class_in_my_namespaced_bundle::'";

    !a_namespace_scoped_class_in_my_namespaced_bundle::
      "In $(this.namespace):$(this.bundle) I do not see 'a_namespace_scoped_class_in_my_namespaced_bundle::'";

    MyNamespace:a_bundle_scoped_class_in_my_namespaced_bundle::
      "In $(this.namespace):$(this.bundle) I see 'MyNamespace:a_bundle_scoped_class_in_my_namespaced_bundle::'";

    !MyNamespace:a_bundle_scoped_class_in_my_namespaced_bundle::
      "In $(this.namespace):$(this.bundle) I do not see 'MyNamespace:a_bundle_scoped_class_in_my_namespaced_bundle::'";

    MyNamespace:a_namespace_scoped_class_in_my_namespaced_bundle::
      "In $(this.namespace):$(this.bundle) I see 'MyNamespace:a_namespace_scoped_class_in_my_namespaced_bundle::'";
}

body file control
{
      namespace => "MyNamespace";
}

bundle agent my_bundle
{
  classes:
      "a_bundle_scoped_class_in_my_namespaced_bundle";

      "a_namespace_scoped_class_in_my_namespaced_bundle"
        scope => "namespace";

  reports:

    a_bundle_scoped_class_in_my_namespaced_bundle::
      "In $(this.namespace):$(this.bundle) I see 'a_bundle_scoped_class_in_my_namespaced_bundle::'";

    !a_bundle_scoped_class_in_my_namespaced_bundle::
      "In $(this.namespace):$(this.bundle) I do not see 'a_bundle_scoped_class_in_my_namespaced_bundle::'";

    a_namespace_scoped_class_in_my_namespaced_bundle::
      "In $(this.namespace):$(this.bundle) I see 'a_namespace_scoped_class_in_my_namespaced_bundle::'";

    !a_namespace_scoped_class_in_my_namespaced_bundle::
      "In $(this.namespace):$(this.bundle) I do not see 'a_namespace_scoped_class_in_my_namespaced_bundle::'";

    MyNamespace:a_bundle_scoped_class_in_my_namespaced_bundle::
      "In $(this.namespace):$(this.bundle) I see 'MyNamespace:a_bundle_scoped_class_in_my_namespaced_bundle::'";

    !MyNamespace:a_bundle_scoped_class_in_my_namespaced_bundle::
      "In $(this.namespace):$(this.bundle) I do not see 'MyNamespace:a_bundle_scoped_class_in_my_namespaced_bundle::'";

    MyNamespace:a_namespace_scoped_class_in_my_namespaced_bundle::
      "In $(this.namespace):$(this.bundle) I see 'MyNamespace:a_namespace_scoped_class_in_my_namespaced_bundle::'";
}
```

# Refactored to use a single bundle for reporting

This version reports essentially the same information but removes the
duplicated reports.

It\'s important to remember that you must make unique promises, which is
why this version emits information showing both the bundle that emited
the infomation and the information about where it was called from.
Without this difference, some output would be suppressed from having
already been reported.

:::: captioned-content
::: caption
Example Policy
:::

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both"}
bundle agent __main__
{
  methods:
      "MyNamespace:my_bundle";
      "report" usebundle => report( "$(this.namespace)", "$(this.bundle)", "called with inherit => 'false'" ), inherit => "false";
 }
 bundle agent report(ns, bundlename, comment)
 {
  reports:
    a_bundle_scoped_class_in_my_namespaced_bundle::
      "In $(this.namespace):$(this.bundle) From $(ns):$(bundlename) $(comment) I see 'a_bundle_scoped_class_in_my_namespaced_bundle::'";
    !a_bundle_scoped_class_in_my_namespaced_bundle::
      "In $(this.namespace):$(this.bundle) From $(ns):$(bundlename) $(comment) I do not see 'a_bundle_scoped_class_in_my_namespaced_bundle::'";
    a_namespace_scoped_class_in_my_namespaced_bundle::
      "In $(this.namespace):$(this.bundle) From $(ns):$(bundlename) $(comment) I see 'a_namespace_scoped_class_in_my_namespaced_bundle::'";
    !a_namespace_scoped_class_in_my_namespaced_bundle::
      "In $(this.namespace):$(this.bundle) From $(ns):$(bundlename) $(comment) I do not see 'a_namespace_scoped_class_in_my_namespaced_bundle::'";
    MyNamespace:a_bundle_scoped_class_in_my_namespaced_bundle::
      "In $(this.namespace):$(this.bundle) From $(ns):$(bundlename) $(comment) I see 'MyNamespace:a_bundle_scoped_class_in_my_namespaced_bundle::'";
    !MyNamespace:a_bundle_scoped_class_in_my_namespaced_bundle::
      "In $(this.namespace):$(this.bundle) From $(ns):$(bundlename) $(comment) I do not see 'MyNamespace:a_bundle_scoped_class_in_my_namespaced_bundle::'";
    MyNamespace:a_namespace_scoped_class_in_my_namespaced_bundle::
      "In $(this.namespace):$(this.bundle) From $(ns):$(bundlename) $(comment) I see 'MyNamespace:a_namespace_scoped_class_in_my_namespaced_bundle::'";
}
body file control
{
      namespace => "MyNamespace";
}
bundle agent my_bundle
{
  classes:
      "a_bundle_scoped_class_in_my_namespaced_bundle";
      "a_namespace_scoped_class_in_my_namespaced_bundle"
        scope => "namespace";
     methods:
      "report" usebundle => default:report( "$(this.namespace)", "$(this.bundle)", "called with inherit => 'false'" ), inherit => "false";
      "report" usebundle => default:report( "$(this.namespace)", "$(this.bundle)", "called with inherit => 'true'" ), inherit => "true";
}
```
::::

``` example
R: In default:report From MyNamespace:my_bundle called with inherit => 'false' I do not see 'a_bundle_scoped_class_in_my_namespaced_bundle::'
R: In default:report From MyNamespace:my_bundle called with inherit => 'false' I do not see 'a_namespace_scoped_class_in_my_namespaced_bundle::'
R: In default:report From MyNamespace:my_bundle called with inherit => 'false' I do not see 'MyNamespace:a_bundle_scoped_class_in_my_namespaced_bundle::'
R: In default:report From MyNamespace:my_bundle called with inherit => 'false' I see 'MyNamespace:a_namespace_scoped_class_in_my_namespaced_bundle::'
R: In default:report From MyNamespace:my_bundle called with inherit => 'true' I see 'a_bundle_scoped_class_in_my_namespaced_bundle::'
R: In default:report From MyNamespace:my_bundle called with inherit => 'true' I do not see 'a_namespace_scoped_class_in_my_namespaced_bundle::'
R: In default:report From MyNamespace:my_bundle called with inherit => 'true' I see 'MyNamespace:a_bundle_scoped_class_in_my_namespaced_bundle::'
R: In default:report From MyNamespace:my_bundle called with inherit => 'true' I see 'MyNamespace:a_namespace_scoped_class_in_my_namespaced_bundle::'
R: In default:report From default:main called with inherit => 'false' I do not see 'a_bundle_scoped_class_in_my_namespaced_bundle::'
R: In default:report From default:main called with inherit => 'false' I do not see 'a_namespace_scoped_class_in_my_namespaced_bundle::'
R: In default:report From default:main called with inherit => 'false' I do not see 'MyNamespace:a_bundle_scoped_class_in_my_namespaced_bundle::'
R: In default:report From default:main called with inherit => 'false' I see 'MyNamespace:a_namespace_scoped_class_in_my_namespaced_bundle::'
```

# References

- [CFEngine Examples](id:38277465-771a-4db4-983a-8dfd434b1aff)
