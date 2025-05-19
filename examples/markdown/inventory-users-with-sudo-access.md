``` {.cfengine3 include-stdlib="t" verbose-mode="nil" inform-mode="nil" exports="both"}
bundle agent main
{
  vars:
#      =lastlog[nickanderson]=Sat Mar 30 19:44:14 -0500 2019

      "users" slist => { "nickanderson", "a10042", "nobody", "billnye" };


  commands:
      `/usr/bin/lastlog | awk '!/(^Username\W+Port\W+From\W+Latest)|(**Never logged in**)/{print ^context=$(this.bundle)$(const.n)"=lastlog["$1"]="$(NF-5), $(NF-4), $(NF-3), $(NF-2), $(NF-1), $NF }' > $(sys.statedir)/lastlog.module.cache`
        contain => in_shell;

  reports:

      "$(users) has sudo access"
        if => regcmp( ".*User $(users) may run.*",
                      execresult( "/usr/bin/sudo -l -U $(users)", noshell));

      # Watch out, unknown users need special handling 
      "$(users) doesn't have sudo access"
        if => regcmp( "(User $(users) is not allowed.*)|(.*unknown user.*)",
                      execresult( "/usr/bin/sudo -l -U $(users) 2>&1", useshell));
}
```

When run as root on my system I get these results:

``` example
R: nickanderson has sudo access
R: a10042 doesn't have sudo access
R: nobody doesn't have sudo access
R: billnye doesn't have sudo access
```

- Yes, nickanderson does have sudo access
- No, a10042 does not have sudo access
- No, nobody doesn\'t have sudo access
- billnye isn\'t a valid user

``` example
root@nickanderson-ThinkPad-W550s:~# sudo -l -U nickanderson
Matching Defaults entries for nickanderson on nickanderson-ThinkPad-W550s:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User nickanderson may run the following commands on nickanderson-ThinkPad-W550s:
    (ALL : ALL) ALL
root@nickanderson-ThinkPad-W550s:~# sudo -l -U a10042
User a10042 is not allowed to run sudo on nickanderson-ThinkPad-W550s.
root@nickanderson-ThinkPad-W550s:~# sudo -l -U nobody
User nobody is not allowed to run sudo on nickanderson-ThinkPad-W550s.
root@nickanderson-ThinkPad-W550s:~# sudo -l -U billnye
sudo: unknown user: billnye
```

``` {.cfengine3 include-stdlib="t" verbose-mode="nil" inform-mode="nil" exports="both"}
bundle agent inventory_lastlog
{
  vars:

      #"users" slist => { "nickanderson", "a10042", "nobody", "billnye" };
      "refresh_frequency"
        string => "30",
        comment => "https://docs.cfengine.com/docs/3.12/reference-promise-types.html#ifelapsed";

      "lastlog_users"
        slist => getindices( lastlog ),
        meta => { "inventory", "attribute_name=Users who have logged in" };

      "users" slist => { @(lastlog_users) };

  commands:

      # Write entries for users who have logged in to a file in the module
      # protocol format, we get the inventory loaded into the agent by a
      # lightweight call to cat and we use promise locks to prevent a call to
      # the shell, lastlog, and awk each agent run.

      # Example: entry
      # ^context=$(this.bundle)
      # ^meta=inventory,attribute_name=Last logged in nickanderson
      # =lastlog[nickanderson]=Sat Mar 30 19:44:14 -0500 2019

      `/usr/bin/lastlog | awk '!/(^Username\W+Port\W+From\W+Latest)|(**Never logged in**)/{print "^context=$(this.bundle)\n^meta=inventory,attribute_name=Last logged in "$1"\n=lastlog["$1"]="$(NF-5), $(NF-4), $(NF-3), $(NF-2), $(NF-1), $NF }' > $(sys.statedir)/lastlog.module.cache`
        contain => in_shell,
        action => if_elapsed( $(refresh_frequencey) );

      `/bin/cat $(sys.statedir)/lastlog.module.cache`
        module => 'true';
}

bundle agent inventory_sudoers( users, refresh_frequency, inventory_prefix )
# @param users A list of users to check for sudo access
# @note Consider getting users by parsing /etc/passed, or from recently logged in users
# Example Cache: $(sys.statedir)/lastlog_users_with_sudo.module.cache
# markburgess
# nickanderson
{
  vars:
      "cache"
        string => "$(sys.statedir)/$(with)_users_with_sudo.module.cache.d/",
        with => canonify( $(inventory_prefix) );

      # "has_sudo[$(users)]"
      #   string => "$(users)",
      #   if => regcmp( ".*User $(users) may run.*",
      #                 execresult( "/usr/bin/sudo -l -U $(users)", noshell)),
      #   unless => fileexists( $(cache) ),
      #   meta => { "inventory", "attribute_name=$(inventory_prefix)User with sudo" };

      "users"
        slist => lsdir( $(cache) ),
        meta => { "inventory", "attribute_name=$(inventory_prefix)User with sudo" };
  files:

      # expire the cache if it exists
      "$(cache)/$(users)"
        delete => tidy,
        action => if_elapsed( $(refresh_frequency) );

      # We have a file semaphore for each user who has sudo access
      "$(cache)/$(users)"
        create => true,
        touch => true,
        if => regcmp( ".*User $(users) may run.*",
                      execresult( "/usr/bin/sudo -l -U $(users)", noshell)),
        action => if_elapsed( $(refresh_frequency) );


      # Watch out, unknown users need special handling
      # If we iterate over known users this isn't likely to be an issue.
      # If we base it off lastlog or last information, then possibly well have an entry for a user that no longer exists.

      # "$(users) doesn't have sudo access"
      #   if => regcmp( "(User $(users) is not allowed.*)|(.*unknown user.*)",
      #                 execresult( "/usr/bin/sudo -l -U $(users) 2>&1", useshell));
}


bundle agent __main__
{
  methods:
      "inventory_lastlog";
      "" usebundle => inventory_sudoers( @(inventory_lastlog.users), "Lastlog ", "1" );
      # So, if you have, local users parsed from /etc/passwd in a list, you could inventory it seperately.
      "" usebundle => inventory_sudoers( @(inventory_passwd.users), "/etc/passwd ", "30" );

      # OR, you could source from multiple places combine the lists, and inventory that.
}
```

What abut this?

``` {.cfengine3 dir="/ssh:root@demohub.a10042.cfengine.com:"}
bundle agent inventory_lastlog
{
  vars:

      #"users" slist => { "nickanderson", "a10042", "nobody", "billnye" };
      "refresh_frequency"
        string => "30",
        comment => "https://docs.cfengine.com/docs/3.12/reference-promise-types.html#ifelapsed";

      "lastlog_users"
        slist => getindices( lastlog ),
        meta => { "inventory", "attribute_name=Users who have logged in" };

      "users" slist => { @(lastlog_users) };

  commands:

      # Write entries for users who have logged in to a file in the module
      # protocol format, we get the inventory loaded into the agent by a
      # lightweight call to cat and we use promise locks to prevent a call to
      # the shell, lastlog, and awk each agent run.

      # Example: entry
      # ^context=$(this.bundle)
      # ^meta=inventory,attribute_name=Last logged in nickanderson
      # =lastlog[nickanderson]=Sat Mar 30 19:44:14 -0500 2019

      `/usr/bin/lastlog | awk '!/(^Username\W+Port\W+From\W+Latest)|(**Never logged in**)/{print "^context=$(this.bundle)\n^meta=inventory,attribute_name=Last logged in "$1"\n=lastlog["$1"]="$(NF-5), $(NF-4), $(NF-3), $(NF-2), $(NF-1), $NF }' > $(sys.statedir)/lastlog.module.cache`
        contain => in_shell,
        action => if_elapsed( $(refresh_frequencey) );

      `/bin/cat $(sys.statedir)/lastlog.module.cache`
        module => "true",
        action => immediate;
}
bundle agent inventory_sudoers( users, refresh_frequency, inventory_prefix )
# @param users A list of users to check for sudo access
# @note Consider getting users by parsing /etc/passed, or from recently logged in users
# Example Cache: $(sys.statedir)/lastlog_users_with_sudo.module.cache
# markburgess
# nickanderson
{
  vars:
      "cache"
        string => "$(sys.statedir)/$(with)_users_with_sudo.module.cache.d",
        with => canonify( $(inventory_prefix) );

      # "has_sudo[$(users)]"
      #   string => "$(users)",
      #   if => regcmp( ".*User $(users) may run.*",
      #                 execresult( "/usr/bin/sudo -l -U $(users)", noshell)),
      #   unless => fileexists( $(cache) ),
      #   meta => { "inventory", "attribute_name=$(inventory_prefix)User with sudo" };

      "users"
        slist => lsdir( $(cache), "", false ),
        meta => { "inventory", "attribute_name=$(inventory_prefix)User with sudo" },
        if => isdir( $(cache) );

  files:

      "$(cache)/."
        create => "true";

      # expire the cache if it exists
      "$(cache)/$(users)"
        delete => tidy,
        action => if_elapsed( $(refresh_frequency) );

      # We have a file semaphore for each user who has sudo access
      "$(cache)/$(users)"
        create => "true",
        touch => "true",
        if => regcmp( ".*User $(users) may run.*",
                      execresult( "/usr/bin/sudo -l -U $(users)", noshell)),
        action => if_elapsed( $(refresh_frequency) );


      # Watch out, unknown users need special handling
      # If we iterate over known users this isn't likely to be an issue.
      # If we base it off lastlog or last information, then possibly well have an entry for a user that no longer exists.

      # "$(users) doesn't have sudo access"
      #   if => regcmp( "(User $(users) is not allowed.*)|(.*unknown user.*)",
      #                 execresult( "/usr/bin/sudo -l -U $(users) 2>&1", useshell));
}


bundle agent __main__
{
  methods:
      "" usebundle => inventory_lastlog;
      "" usebundle => inventory_sudoers( @(inventory_lastlog.users), "1", "Lastlog " );
}

body contain in_shell
# @brief run command in shell
#
# **Example:**
#
# ```cf3
#  commands:
#    "/bin/pwd | /bin/cat"
#      contain => in_shell;
# ```
{
      useshell => "true"; # canonical "useshell" but this is backwards-compatible
}


body action if_elapsed(x)
# @brief Evaluate the promise every `x` minutes
# @param x The time in minutes between promise evaluations
{
      ifelapsed => "$(x)";
      expireafter => "$(x)";
}

body delete tidy
# @brief Delete the file and remove empty directories
# and links to directories
{
      dirlinks => "delete";
      rmdirs   => "true";
}

body action immediate
{
      ifelapsed => "0";
}
```

Here is an example run.

``` example
root@nickanderson-ThinkPad-W550s:~# /usr/bin/time cf-agent -f ./t.cf --show-evaluated-vars=default:inventory_ -K
Variable name                            Variable value                                               Meta tags                               
default:inventory_lastlog.lastlog[nickanderson] Sat Mar 30 19:44:14 -0500 2019                               inventory,attribute_name=Last logged in nickanderson,source=module
default:inventory_lastlog.lastlog_users   {"nickanderson"}                                            source=promise,inventory,attribute_name=Users who have logged in
default:inventory_lastlog.refresh_frequency 30                                                           source=promise                          
default:inventory_lastlog.users           {"nickanderson"}                                            source=promise                          
default:inventory_sudoers.cache          /var/cfengine/state/Lastlog__users_with_sudo.module.cache.d  source=promise                          
0.05user 0.01system 0:00.07elapsed 96%CPU (0avgtext+0avgdata 11716maxresident)k
0inputs+168outputs (0major+4239minor)pagefaults 0swaps
```

``` example
[root@demohub ~]# cf-agent -Kf update.cf; cf-agent --show-evaluated-vars="default:inventory_(sudoers|lastlog)" -b main -v | cf-profile -a | egrep "inventory_(sudoers|lastlog)"
|--------> Bundle default:inventory_lastlog
|--------> Bundle default:inventory_sudoers( {"@(inventory_lastlog.users)","1","Lastlog "})
    3.  0.03042s  : inventory_sudoers( {"@(inventory_lastlog.users)","1","Lastlog "})
    5.  0.01710s  : inventory_lastlog
[root@demohub ~]# cf-agent -Kf update.cf; cf-agent --show-evaluated-vars="default:inventory_(sudoers|lastlog)" -b main -v | cf-profile -a | egrep "inventory_(sudoers|lastlog)"
|--------> Bundle default:inventory_lastlog
|--------> Bundle default:inventory_sudoers( {"@(inventory_lastlog.users)","1","Lastlog "})
    2.  0.02551s  : inventory_sudoers( {"@(inventory_lastlog.users)","1","Lastlog "})
    3.  0.01095s  : inventory_lastlog
[root@demohub ~]# cf-agent -Kf update.cf; cf-agent --show-evaluated-vars="default:inventory_(sudoers|lastlog)" -b main -v | cf-profile -a | egrep "inventory_(sudoers|lastlog)"
|--------> Bundle default:inventory_lastlog
|--------> Bundle default:inventory_sudoers( {"@(inventory_lastlog.users)","1","Lastlog "})
    2.  0.02233s  : inventory_sudoers( {"@(inventory_lastlog.users)","1","Lastlog "})
    3.  0.01231s  : inventory_lastlog
```

I made a few tweaks

``` {.cfengine3 include-stdlib="t" verbose-mode="nil" inform-mode="nil" exports="both"}
bundle agent inventory_lastlog(refresh_frequency)
{
  vars:

      "users"
        slist => getindices( lastlog );

      "i_lastlog[$(users)]"
        string => "$(users)",
        meta => { "inventory", "attribute_name=Users who have logged in" };


  commands:

      # Write entries for users who have logged in to a file in the module
      # protocol format, we get the inventory loaded into the agent by a
      # lightweight call to cat and we use promise locks to prevent a call to
      # the shell, lastlog, and awk each agent run.

      # Example: entry
      # ^context=$(this.bundle)
      # ^meta=inventory,attribute_name=Last logged in nickanderson
      # =lastlog[nickanderson]=Sat Mar 30 19:44:14 -0500 2019

      `/usr/bin/lastlog | awk '!/(^Username\W+Port\W+From\W+Latest)|(**Never logged in**)/{print "^context=$(this.bundle)\n^meta=inventory,attribute_name=Last logged in "$1"\n=lastlog["$1"]="$(NF-5), $(NF-4), $(NF-3), $(NF-2), $(NF-1), $NF }' > $(sys.statedir)/lastlog.module.cache`
        contain => in_shell,
        action => if_elapsed( $(refresh_frequencey) );

      `/bin/cat $(sys.statedir)/lastlog.module.cache`
        handle => "load_lastlog_inventory_from_cache",
        module => 'true',
        action => immediate;
}


bundle agent inventory_sudoers( users, refresh_frequency, inventory_prefix )
# @param users A list of users to check for sudo access
# @note Consider getting users by parsing /etc/passed, or from recently logged in users
# Example Cache: $(sys.statedir)/lastlog_users_with_sudo.module.cache
# markburgess
# nickanderson
{
  vars:
      "cache"
        string => "$(sys.statedir)/$(with)_users_with_sudo.module.cache.d",
        with => canonify( $(inventory_prefix) );

      "users"
        slist => lsdir( "$(cache)/", "(?!\.|\.\.).*", false ),
        if => isdir( $(cache) );

      "sudo[$(users)]"
        string => "$(users)",
        meta => { "inventory", "attribute_name=$(inventory_prefix)User with sudo" };

  files:

      "$(cache)/.*"
        delete => tidy,
        action => if_elapsed( $(refresh_frequency) );

      "$(cache)/."
        create => "true";

      # We have a file semaphore for each user who has sudo access
      "$(cache)/$(users)"
        create => "true",
        touch => "true",
        if => regcmp( ".*User $(users) may run.*",
                      execresult( "/usr/bin/sudo -l -U $(users)", noshell)),
        action => if_elapsed( $(refresh_frequency) );

      # Watch out, unknown users need special handling
      # If we iterate over known users this isn't likely to be an issue.
      # If we base it off lastlog or last information, then possibly well have an entry for a user that no longer exists.

      # "$(users) doesn't have sudo access"
      #   if => regcmp( "(User $(users) is not allowed.*)|(.*unknown user.*)",
      #                 execresult( "/usr/bin/sudo -l -U $(users) 2>&1", useshell));
}

bundle agent __main__
{
  methods:
    linux::
      "Inventory users who have logged in"
        usebundle => inventory_lastlog(30), # We refresh the cache from lastlog once every 30 minutes
        action => immediate; # We always do the inventory

      "Inventory Sudoers who have logged in"
        usebundle => inventory_sudoers( @(inventory_lastlog.users), "720", "Lastlog " ), # Once every 12 hours re-check the lastlog users who have sudo access
        action => immediate; # We always do the inventory

      # So, if you have, local users parsed from /etc/passwd in a list, you
      #      could inventory it seperately. "" usebundle => inventory_sudoers(
      #      @(inventory_passwd.users), "/etc/passwd ", "30" );

      # OR, you could source from multiple places combine the lists, and inventory that.

}
```

Here I used cf-profile to get the timings for inventory~lastlog~ and
inventory~sudoers~ bundles.

``` example
[root@demohub ~]# cf-agent -Kf update.cf; cf-agent --show-evaluated-vars="default:inventory_(sudoers|lastlog)" -b main -v | cf-profile -a | egrep "inventory_(sudoers|lastlog)"
|--------> Bundle default:inventory_lastlog( {"30"})
|--------> Bundle default:inventory_sudoers( {"@(inventory_lastlog.users)","720","Lastlog "})
    2.  0.02582s  : inventory_sudoers( {"@(inventory_lastlog.users)","720","Lastlog "})
    3.  0.01207s  : inventory_lastlog( {"30"})
[root@demohub ~]# cf-agent -Kf update.cf; cf-agent --show-evaluated-vars="default:inventory_(sudoers|lastlog)" -b main -v | cf-profile -a | egrep "inventory_(sudoers|lastlog)"
|--------> Bundle default:inventory_lastlog( {"30"})
|--------> Bundle default:inventory_sudoers( {"@(inventory_lastlog.users)","720","Lastlog "})
    2.  0.02570s  : inventory_sudoers( {"@(inventory_lastlog.users)","720","Lastlog "})
    3.  0.00585s  : inventory_lastlog( {"30"})
[root@demohub ~]# cf-agent -Kf update.cf; cf-agent --show-evaluated-vars="default:inventory_(sudoers|lastlog)" -b main -v | cf-profile -a | egrep "inventory_(sudoers|lastlog)"
|--------> Bundle default:inventory_lastlog( {"30"})
|--------> Bundle default:inventory_sudoers( {"@(inventory_lastlog.users)","720","Lastlog "})
    2.  0.02588s  : inventory_sudoers( {"@(inventory_lastlog.users)","720","Lastlog "})
    3.  0.01013s  : inventory_lastlog( {"30"})
```

The test isn\'t statistically significant (I don\'t have many users or
sudoers, nor are my test systems old where large lastlog files could
have an impact on speed).
