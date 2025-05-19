``` {.cfengine3 include-stdlib="t" log-level="info" exports="both" tangle="install_ntpd_and_configure_driftfile.cf"}
bundle agent excercise_configure_nptd_driftfile
{
  vars:
      "config[driftfile]" string => "/var/lib/ntp/drift";
  packages:
      "ntp" policy => "present";
  services:
      "ntpd" service_policy => "start";
      "ntpd"
        service_policy => "restart",
        if => "ntpd_conf_repaired";
  files:
      "/etc/ntp.conf"
        edit_line => set_line_based("$(this.bundle).config", # Key Values
                                    " ",        # Separator
                                    "\s+",      # Separator regex
                                    ".*",       # Keys to use
                                    "\s*#\s*"), # Ignore lines
        classes => results("bundle", "ntpd_conf");
}
bundle agent __main__{methods:"excercise_configure_nptd_driftfile";}
```
