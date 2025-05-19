We query the [Host API](id:7986c727-30f5-4182-9b89-b1c383359d9c) and
munge the data wit `jq` to feed cf-runagent a list of hosts to run on.

``` {.bash org-language="sh" results="output" exports="both"}
exec 2>&1
cf-runagent --background=3 --remote-bundles satellite_registration \
-H $(curl -s -k --user $USER:$PASSWORD \
      https://$HUB/api/host\?count\=2\&context-include\=not_rh_satellite_registered |\
      jq '.data | map(.ip) | join(",")'|tr -d '"')
:
```

The [Actions API](id:019a92bc-ed7f-4d9d-98e1-fb2e48e777a7) can be used
to trigger agent policy runs but as of 3.18.2 does not have the ability
to specify any options like bundlesequence or define additional classes.
