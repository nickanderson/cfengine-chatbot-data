:::: captioned-content
::: caption
Example namespace declaration
:::

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both" tangle="declaring_namespaces.cf"}
# By default we are in the default namespace

bundle agent __main__
{
  methods:
      "Main in My_Namespace namespace"
        usebundle => My_Namespace:main;

      "Main in your_namespace namespace"
        usebundle => your_namespace:main;

      "my_bundle in default namespace"
        usebundle => my_bundle;

  reports:
      "Inside $(this.namespace):$(this.bundle)";
}

body file control
# From here until the next namespace declaration all bundles and bodies are
# defined in My_Namespace.
{
      namespace => "My_Namespace";
}

bundle agent main
{
  reports:
      "Inside $(this.namespace):$(this.bundle)";
}

body file control
# From here until the next namespace declaration all bundles and bodies are
# defined in your_namespace.
{
        namespace => "your_namespace";
}

bundle agent main
{
  reports:
      "Inside $(this.namespace):$(this.bundle)";
}

# From here until the next namespace declaration we return to the default namespace.
body file control
{
        namespace => "default";
}

bundle agent my_bundle
{
  reports:
      "Inside $(this.namespace):$(this.bundle)";
}

```
::::

``` example
R: Inside My_Namespace:main
R: Inside your_namespace:main
R: Inside default:my_bundle
R: Inside default:main
```

**References:**

- [CFEngine Examples](id:38277465-771a-4db4-983a-8dfd434b1aff)
