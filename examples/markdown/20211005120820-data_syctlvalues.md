``` {.cfengine3 tangle="data_syctlvalues.cf"}
bundle agent inventory_sysctl
{
  meta:
      "tags" slist => { "autorun" };

  vars:
      "_sysctl_d" data => data_sysctlvalues();

      "_sysctl_i"
        slist => getindices( "_sysctl_d" );

      # Define an individual variable for each item
      "sysctl[$(_sysctl_i)]"
        string => "$(_sysctl_d[$(_sysctl_i)])",
        meta => { "inventory", "attribute_name=Kernel Tunable $(_sysctl_i)" };

  reports:
      "data_sysctlvalues() returned:$(with)"
        with => storejson( @(_sysctl_d) );

}
bundle agent __main__
{
  methods: "inventory_sysctl";

}
```
