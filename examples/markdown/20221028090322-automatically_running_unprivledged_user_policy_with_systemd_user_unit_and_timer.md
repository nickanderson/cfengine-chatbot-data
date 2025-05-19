``` {.bash org-language="sh" results="output" exports="both"}
exec 2>&1
mkdir -p ~/.config/systemd/user/
:
```

``` {.conf tangle="~/.config/systemd/user/cf-agent.service"}
[Unit]
Description=CFEngine cf-agent user service

[Service]
Type=oneshot
ExecStart=%h/.cfagent/bin/cf-agent
```

``` {.conf tangle="~/.config/systemd/user/cf-agent.timer"}
[Unit]
Description=CFEngine cf-agent user timer

[Timer]
OnBootSec=2m
OnUnitActiveSec=5m
Unit=cf-agent.service

[Install]
WantedBy=timers.target
```

:::: captioned-content
::: caption
Shell command with stderr
:::

``` {.bash org-language="sh" results="output" exports="both"}
exec 2>&1
if [ ! -e ~/.cfagent/inputs/promises.cf ]; then
  echo "bundle agent __main__
{
  reports:
    inform_mode::
      'Hello World';
}" > ~/.cfagent/inputs/promises.cf
fi
chmod 600 ~/.cfagent/inputs/promises.cf
systemctl --user enable cf-agent
systemctl --user restart cf-agent
systemctl --user enable cf-agent.timer
systemctl --user restart cf-agent.timer
:
```
::::

``` example
The unit files have no installation config (WantedBy=, RequiredBy=, Also=,
Alias= settings in the [Install] section, and DefaultInstance= for template
units). This means they are not meant to be enabled using systemctl.

Possible reasons for having this kind of units are:
• A unit may be statically enabled by being symlinked from another unit's
  .wants/ or .requires/ directory.
• A unit's purpose may be to act as a helper for some other unit which has
  a requirement dependency on it.
• A unit may be started when needed via activation (socket, path, timer,
  D-Bus, udev, scripted systemctl call, ...).
• In case of template units, the unit is meant to be enabled with some
  instance name specified.
```
