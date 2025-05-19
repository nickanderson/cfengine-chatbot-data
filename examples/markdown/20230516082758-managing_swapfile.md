``` {.cfengine3 tangle="managing_swapfile.cf"}
bundle agent swap_available
{
# Mysql behaves badly without any swap on a system

  vars:
      "swapfile" string => "/swapfile1";
      "swapsizeM" string => "1024";

  classes:
      "swapfile_active" expression => returnszero("$(paths.grep) '/$(swapfile) ' /proc/swaps", "noshell");
      "swapfile_exists" expression => fileexists($(swapfile));

  files:
    "$(swapfile)"
      perms => mog("0600", "root", "root");

  commands:
    !swapfile_exists::
      "/bin/dd"
        args => "if=/dev/zero of=$(swapfile) bs=1M count=$(swapsizeM)",
        classes => scoped_classes_generic("bundle", "swapfile_creation");

    swapfile_creation_ok|swapon_not_kept::
      "/sbin/mkswap"
        args => "$(swapfile)",
        comment => "We need to mkswap in order for the swapfile to be usable";

    !swapfile_active::
      "/sbin/swapon"
        args => "--fixpgsz $(swapfile)",
        classes => scoped_classes_generic("bundle", "swapon");
}
```
