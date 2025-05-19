> This works but will give an error on all hosts where this augments
> file doesn\'t exist.

``` json
{
    "$(sys.inputdir)/augments/$(sys.uqhost).json": {
        "variables": {
            "cfengine_enterprise_federation:postgres_config.shared_buffers": {
                "value": "2560MB"
            },
            "cfengine_enterprise_federation:postgres_config.max_wal_size": {
                "value": "20480MB"
            },
            "cfengine_enterprise_federation:postgres_config.checkpoint_timeout": {
                "value": "30min"
            }
        }
    },
    "augments": [
        "$(sys.inputdir)/augments/$(sys.uqhost).json"
    ]
}
```

Right, you should expect to get errors like
`error: Could not load requested further augments from file '$(sys.uqhost).json'`{.verbatim}
if the file does not exist for a given host.

> I see you support augments file,
> \$(sys.workdir)/data/host~specific~.json, but this is outside of
> source control support. What was the thought process for doing that?

Right, `$(sys.workdir)/data/host_specific.json`{.verbatim} is outside of
the policy, so it can change at a different rate. It\'s true that it\'s
not version controlled, but you could have something make the directory
a git repo and commit and push all the changes automatically. Something
like that might eventually come by default.

If you don\'t want to use `host_specific.json`{.verbatim} as distributed
from the hub, you can still achieve a host specific data load without
errors by using a stable filename and making sure that it has valid
json.

For example:

``` json
{
    "$(sys.inputdir)/augments/$(sys.uqhost).json": {
        "variables": {
            "cfengine_enterprise_federation:postgres_config.shared_buffers": {
                "value": "2560MB"
            },
            "cfengine_enterprise_federation:postgres_config.max_wal_size": {
                "value": "20480MB"
            },
            "cfengine_enterprise_federation:postgres_config.checkpoint_timeout": {
                "value": "30min"
            }
        }
    },
    "augments": [
        "$(sys.workdir)/augments/custom-host-specific.json"
    ]
}
```

Then, in your update policy you can take care of seeding that data.

In this contrived example we expect there is a data directory next to
the policy file and inside that data directory there are files for
specific hosts named for the unqualified hostname. The policy simply
makes sure that the right host specific file is in place and if no host
specific data exists, then lay down a valid json file with no data.

:::: captioned-content
::: caption
Example Policy
:::

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both"}
bundle agent custom_host_specific_data
{
  vars:
    "my_custom_host_specific_data"
      string => "$(this.promise_dirname)/data/$(sys.uqhost).json";

   files:
    "$(sys.workdir)/augments/." create => "true";

    "$(sys.workdir)/augments/custom-host-specific.json"
      if => fileexists( $(my_custom_host_specific_data) ),
      copy_from => local_dcp( $(my_custom_host_specific_data) ),
      comment => "If we have host speific data, we want it in place for use";

    "$(sys.workdir)/augments/custom-host-specific.json"
      if => not( fileexists( $(my_custom_host_specific_data) ) ),
      content => "{}",
      comment => "If we do not have host speific data, we want to place an empty set to avoid errors";
}
```
::::

Does that make sense?
