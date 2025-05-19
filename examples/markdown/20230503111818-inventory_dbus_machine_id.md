:::: captioned-content
::: caption
Example custom inventory for DBUS machine id
:::

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both" tangle="inventory_dbus_machine_id.cf"}
bundle agent inventory_dbus_machine_id
{
  vars:
    "dbus_machine_id_file"
        string => "/var/lib/dbus/machine-id";

      "dbus_machine_id" -> { "Compliance:1123" }
        string => readfile( $(dbus_machine_id_file) ),
        if => fileexists( $(dbus_machine_id_file) ),
        meta =>  { "inventory", "attribute_name=DBUS machine id" };

}

bundle agent __main__
{
  methods:
      "inventory_dbus_machine_id";

  reports:
      "DBUS machine id: $(inventory_dbus_machine_id.dbus_machine_id)";
}
```
::::

``` example
R: DBUS machine id: 05207246c07e4cb8b288f8f1d4cf01ff
```

Here is the example adjusted to use simple caching.

:::: captioned-content
::: caption
Example custom inventory for DBUS machine id with caching
:::

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both"}
bundle agent inventory_dbus_machine_id
{
  vars:
      "dbus_machine_id_file"
        string => "/var/lib/dbus/machine-id";

      "dbus_machine_id_cache"
        string => "$(sys.statedir)/$(this.bundle).dat";

      "dbus_machine_id" -> { "Compliance:1123" }
        string => readfile( $(dbus_machine_id_cache) ),
        if => fileexists( $(dbus_machine_id_cache) ),
        meta =>  { "inventory", "attribute_name=DBUS machine id" };

  commands:
      "$(paths.cat) $(dbus_machine_id_file) > $(dbus_machine_id_cache)"
        contain => default:in_shell_and_silent;

  files:
      # Let's clear the cache at least once a day (or when someone runs cf-agent without locks)
      "$(dbus_machine_id_cache)"
        delete => default:tidy,
        action => default:if_elapsed_day;

}

bundle agent __main__
{
  methods:
      "inventory_dbus_machine_id";

  reports:
      "DBUS machine id: $(inventory_dbus_machine_id.dbus_machine_id)";
}
```
::::

``` example
    info: Executing 'no timeout' ... '/bin/cat /var/lib/dbus/machine-id > /home/nickanderson/.cfagent/state/inventory_dbus_machine_id.dat'
    info: Completed execution of '/bin/cat /var/lib/dbus/machine-id > /home/nickanderson/.cfagent/state/inventory_dbus_machine_id.dat'
R: DBUS machine id: 05207246c07e4cb8b288f8f1d4cf01ff
```
