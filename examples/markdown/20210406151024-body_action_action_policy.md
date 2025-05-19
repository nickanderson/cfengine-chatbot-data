``` {.cfengine3 tangle="body_action_action_policy.cf"}
bundle agent __main__
{
  files:
      "/tmp/file-action-default"
        content => "Hello world from /tmp/file-action-default";

      "/tmp/file-action-warn"
        create => "true",
        content => "Hello world from /tmp/file-action-warn",
        action => policy( "warn" );
}

body action policy( action_policy )
{
      action_policy => "$(action_policy)";
}
```

What happens when you run the policy?

:::: captioned-content
::: caption
:::

``` {.bash org-language="sh" results="output" exports="both"}
exec 2>&1
cf-agent --no-lock --log-level inform --file ./body_action_action_policy.cf
:
```
::::

Notice the output indicates `/tmp/file-action-default`{.verbatim} was
created and `/tmp/file-action-warn`{.verbatim} was not created (because
of the action policy).

```{=org}
#+RESULTS:
```
``` example
    info: Updated content of '/tmp/file-action-default' with content 'Hello world from /tmp/file-action-default'
 warning: No action was requested for file '/tmp/file-action-default'. Maybe a typo in the policy?
    info: files promise '/tmp/file-action-default' repaired
 warning: Should create file '/tmp/file-action-warn', mode '0600', but only warning promised
 warning: Warnings encountered when actuating files promise '/tmp/file-action-warn'
```
