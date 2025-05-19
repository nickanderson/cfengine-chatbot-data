``` {.cfengine3 tangle="the_first_entry_in_sys_interfaces_matches_sys_ipv4.cf"}
bundle agent main
{
  vars:
    "first_iface" string => nth( "sys.interfaces", 0);

  reports:

    # Here we report if the IP address of the FIRST INTERFACE listed in
    # sys.interfaces matches the IP of sys.ipv4

    "$(sys.ipv4[$(first_iface)]) == $(sys.ipv4)"
      if => strcmp( $(sys.ipv4), "$(sys.ipv4[$(first_iface)])" );

    "$(sys.ipv4[$(first_iface)]) != $(sys.ipv4)"
      if => not( strcmp( $(sys.ipv4), "$(sys.ipv4[$(first_iface)])" ) );
}
```
