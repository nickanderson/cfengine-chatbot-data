``` {.cfengine3 include-stdlib="t" log-level="info" exports="both" tangle="locking_down_gnome_3_screensaver_settings.cf"}
bundle agent __main__
{

   methods:
     "inventory_gnome3";
     "manage_gnome3_screensaver";

}
bundle agent inventory_gnome3
{
  vars:

      # I think it's usually a good idea to look before you leap, inventory back
      # key settings that you think you want to manage so that you can report on
      # things and see where you are before you start enforcing.

    "file" string => "/etc/dconf/profile/user";


      # This one might not be super useful, but after you report on it, you can simply delete the promise to stop reporting it.

    "etc_dconf_profile_user"
      string => readfile( "$(file)" ),
      meta => { "inventory", "attribute_name=Gnome3 dconf profile user"},
      if => fileexists("$(file)");

      # I think that the files in /etc/dconf/db/local.d would be good to inventory
      # "etc_dconf_db_local_d_files" slist => lsdir( "/etc/dconf/db/local.d", ".*", "no" );

      # And, for those files (or some of them) it could be very useful to parse
      # out and inventory key settings. For example, the value of idle-delay,
      # lock-enabled, lock-delay, picture-uri in 00-screensaver. Remember,
      # CFEngine inventory is not for large data, probably don't want to
      # inventory things that are >1k.

}
bundle agent manage_gnome3_screensaver
{
  vars:
      # In augments for example:
      # { "variables": { "default:manage_gnome3_screensaver.timeout": { "value": "300" }}}
      "timeout"
        string => "600",
        if => not( isvariable( "timeout" ));

  files:
    #https://help.gnome.org/admin/system-admin-guide/stable/desktop-lockscreen.html.en
      # If we know the FULL content, we should manage the full content, but since I dunno
      # Adam says: the file exists, user-db:user, system-db:local
    "/etc/dconf/profile/user"
      perms => default:mog( "root", "root", "644" ),
      classes => results( "bundle", "_etc_dconf_profile_user" ),
      content => "# Managed by CFEngine
user-db:user
system-db:local";

     "/etc/dconf/db/local.d/."
      perms => default:mog( "root", "root", "755" );

     "/etc/dconf/db/local.d/00-screensaver"
      perms => default:mog( "root", "root", "644" ),
      classes => results( "bundle", "_etc_dconf_db_local_d_00_screensaver" ),
      content => "# Managed by CFEngine
[org/gnome/desktop/session]

# Set the lock time out in seconds before the session is considered idle
#idle-delay=uint32 600
idle-delay=uint32 $(data:gnome_screensaver.idle_delay_seconds)

[org/gnome/desktop/screensaver]

# Set this to true to lock the screen when the screensaver activates
lock-enabled=true

# Lock screen after screen turns off
lock-delay=uint32 0

# Specify the path to the desktop background image file
picture-uri='file:///usr/share/backgrounds/tusimple.png'

# Specify one of the";

    "/etc/dconf/db/local.db/locks/screensaver"
      #classes => results( "bundle", "_etc_dconf_db_local_db_locks_screensaver" ),
      classes => results( "bundle", "$(this.promiser)" ), # This will canonify to the commented out version above
      perms => default:mog( "root", "root", "644" ),
      content => "# Managed by CFEngine
# Lock desktop screensaver settings
/org/gnome/desktop/session/idle-delay
/org/gnome/desktop/screensaver/lock-delay
";

  commands:
      "/usr/bin/dconf update"
        # If any one of the key files has been repaired, then we need to update dconf
        if => or(
                   concat( canonify( "/etc/dconf/db/local.db/locks/screensaver" ), "_repaired" ),
                   concat( canonify( "/etc/dconf/db/local.d/00-screensaver"), "_repaired"),
                   concat( canonify( "/etc/dconf/profile/user" ), "_repaired") );


}
```
