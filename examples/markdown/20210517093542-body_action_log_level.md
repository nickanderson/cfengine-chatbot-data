[CFEngine Examples](id:38277465-771a-4db4-983a-8dfd434b1aff)

`log_level`{.verbatim} in *action bodies* specifies the reporting level
sent to `syslog`. Action bodies can be attached to *any* promise type,
thus, this can help to control the information logged to syslog for
individual promises.

``` {.cfengine3 tangle="body_action_log_level.cf"}
bundle agent __main__
{
  files:
    "/tmp/example_file.txt"
      create => "true",
      touch => "true",
      action => to_syslog( "inform" ); # (inform|verbose|(error*|log))

}
body action to_syslog( log_level )
{
    log_level => "$(log_level)";
}
```

:::: captioned-content
::: caption
View syslog entries
:::

``` {.bash org-language="sh" results="output" exports="both"}
exec 2>&1
grep "/tmp/example_file.txt" /var/log/syslog
:
```
::::

``` example
May 17 09:41:34 nickanderson-ThinkPad-W550s cf-agent[281305]: CFEngine(agent)  Touched (updated time stamps) for path '/tmp/example_file.txt'
May 17 09:41:34 nickanderson-ThinkPad-W550s cf-agent[281305]: CFEngine(agent)  files promise '/tmp/example_file.txt' repaired
May 17 09:44:06 nickanderson-ThinkPad-W550s cf-agent[281433]: CFEngine(agent)  Touched (updated time stamps) for path '/tmp/example_file.txt'
May 17 09:44:06 nickanderson-ThinkPad-W550s cf-agent[281433]: CFEngine(agent)  files promise '/tmp/example_file.txt' repaired
```
