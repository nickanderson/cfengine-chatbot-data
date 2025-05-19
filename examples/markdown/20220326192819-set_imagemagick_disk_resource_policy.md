:::: captioned-content
::: caption
Sample ImageMagick Policy
:::

``` {.xml tangle="/tmp/example.xml"}
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE policymap [
  <!ELEMENT policymap (policy)*>
  <!ATTLIST policymap xmlns CDATA #FIXED ''>
  <!ELEMENT policy EMPTY>
  <!ATTLIST policy xmlns CDATA #FIXED '' domain NMTOKEN #REQUIRED
    name NMTOKEN #IMPLIED pattern CDATA #IMPLIED rights NMTOKEN #IMPLIED
    stealth NMTOKEN #IMPLIED value CDATA #IMPLIED>
]>
<policymap>
  <policy domain="resource" name="memory" value="256MiB"/>
  <policy domain="resource" name="map" value="512MiB"/>
  <policy domain="resource" name="width" value="16KP"/>
  <policy domain="resource" name="height" value="16KP"/>
  <!-- <policy domain="resource" name="list-length" value="128"/> -->
  <policy domain="resource" name="area" value="128MP"/>
  <!--<policy domain="resource" name="disk" value="1GiB"/> -->
  <policy domain="resource" name="disk" value="20GiB"/>
</policymap>
```
::::

``` {.bash org-language="sh" results="output" exports="both"}
exec 2>&1
xmllint --xpath "/policymap/policy[@name='disk'][@domain='resource']" /tmp/example.xml
:
```

``` example
<policy xmlns="" domain="resource" name="disk" value="1GiB"/>
```

``` cfengine3
bundle agent __main__
# @brief Drive default bundlesequence if called as policy entry
{
  methods:
    "ImageMagick";
}
bundle agent ImageMagick()
# @brief ImageMagick manages configuration for ImageMagick ...
{
  files:
     "/tmp/example.xml"
       edit_xml => example;
}
 bundle edit_xml example
 {
  build_xpath:
    "/policymap";

  set_attribute:
    "value"
         attribute_value => "200GiB",
         select_xpath    => "/policymap/policy[@name='disk'][@domain='resource']";
 }

```

```{=org}
#+RESULTS:
```
``` {.bash org-language="sh" results="output" exports="both"}
exec 2>&1
xmllint --xpath "/policymap/policy[@name='disk'][@domain='resource']" /tmp/example.xml
:
```

``` example
<policy xmlns="" domain="resource" name="disk" value="200GiB"/>
```

**References:**

- [CFEngine Examples](id:38277465-771a-4db4-983a-8dfd434b1aff)

\-[files: edit~xml~](id:043a128f-3f63-44d3-881c-7dccc7b40339)
