Use the `sys.ip2iface`{.verbatim} map.

``` {.txt tangle="/tmp/example.tpl"}
### Managed by CFEngine
$(sys.ipv4) is on $(sys.ip2iface[$(sys.ipv4)])
```

:::: captioned-content
::: caption
Example Policy
:::

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both" tangle="rendering_a_file_that_contains_interface_name_using_classic_cfengine_template.cf"}
bundle agent __main__
{
  files:
      "/tmp/example.txt"
        create => "true", # This will ensure the file is present, if template expansion fails, you might have an empty file
        template_method => "cfengine", # This is the default, but not necessarily preffered
        edit_template => "/tmp/example.tpl";
  reports:
    "/tmp/example.txt"
      printfile => cat( $(this.promiser) );
}
```
::::

``` example
    info: Created file '/tmp/example.txt', mode 0600
    info: Inserted the promised line '### Managed by CFEngine' into '/tmp/example.txt' after locator
    info: insert_lines promise '### Managed by CFEngine' repaired
    info: Inserted the promised line '192.168.42.232 is on wlp0s20f3' into '/tmp/example.txt' after locator
    info: insert_lines promise '192.168.42.232 is on wlp0s20f3' repaired
    info: Edited file '/tmp/example.txt'
R: /tmp/example.txt
R: ### Managed by CFEngine
R: 192.168.42.232 is on wlp0s20f3
```
