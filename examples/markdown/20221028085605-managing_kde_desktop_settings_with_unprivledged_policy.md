Have [systemd timer setup to run user
policy](id:580fec8c-620c-43fe-995c-c0f5e26eb97b), then write policy like
anything else.

:::: captioned-content
::: caption
Example Policy
:::

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both" tangle="managing_kde_desktop_settings_with_unprivledged_policy.cf"}
bundle agent __main__
{
  vars:
    "desired_screensaver_autolock" string => "false";
    "current_screensaver_autolock" string => execresult( "/usr/bin/kreadconfig5 --file kscreenlockerrc --group Daemon --key Autolock", noshell );

  commands:
    "/usr/bin/kwriteconfig5"
      arglist => {
                 "--file",
                   "kscreenlockerrc",
                   "--group",
                   "Daemon",
                   "--key",
                   "Autolock",
                   "false"
      },
      if => strcmp( "$(current_screensaver_autolock)", "true" );


}
```
::::

``` example
    info: Executing 'no timeout' ... '/usr/bin/kwriteconfig5 --file kscreenlockerrc --group Daemon --key Autolock false'
    info: Completed execution of '/usr/bin/kwriteconfig5 --file kscreenlockerrc --group Daemon --key Autolock false'
```
