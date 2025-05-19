``` {.cfengine3 tangle="inventorying_systemd_services.cf"}
bundle agent inventory_systemd_services
{
  meta:
    systemd::
      "tags" slist => { "autorun" };

  vars:

      "cache" string => "$(sys.statedir)/systemd-units-running.dat";

      "s"
        slist => readstringlist( $(cache), "", $(const.n), inf, inf),
        if => fileexists( $(cache) );

@if minimum_version(3.12)
      "i[$(s)]"
        string => "$(with)",
        with => nth( string_split( $(s), "\.service", inf ), 0 ),
        meta => { "inventory", "attribute_name=Systemd services running"};
@endif

  commands:

      "$(paths.systemctl) list-units --type=service --state=running | awk '/^UNIT .*/{flag=1;next}/^$/{flag=0}flag{print $1}' > $(cache)"
        contain => in_shell;

}
bundle agent __main__
{
      methods: "inventory_systemd_services";
}
```
