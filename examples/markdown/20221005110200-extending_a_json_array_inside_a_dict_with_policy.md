``` json
{
    "hosts": {
        "default": [
            "host005"
        ],
        "policy_hub": [
            "hub"
        ],
        "web1": [
            "host002",
            "host003"
        ],
        "web3": [
            "host002"
        ]
    }
}

```

:::: captioned-content
::: caption
Example Policy
:::

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both" tangle="extending_a_json_array_inside_a_dict_with_policy.cf"}
bundle agent __main__
{
  vars:
      "hosts_data" data  => readjson("/tmp/hosts.json");
      "k_group"         slist => getindices("hosts_data[hosts]");
      "_hosts_data[hosts][$(k_group)]" slist => getvalues("hosts_data[hosts][$(k_group)]");
      "_hosts_data[hosts][default]" slist => {getvalues("hosts_data[hosts][default]"),"my_server" };

  reports:
      "$(with)" with => storejson( mergedata( _hosts_data ) );
}
```
::::

``` example
R: {
  "hosts": {
    "default": [
      "host005",
      "my_server"
    ],
    "policy_hub": [
      "hub"
    ],
    "web1": [
      "host002",
      "host003"
    ],
    "web3": [
      "host002"
    ]
  }
}
```
