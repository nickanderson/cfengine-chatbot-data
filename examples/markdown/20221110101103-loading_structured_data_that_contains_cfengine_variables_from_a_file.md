When you load data from a file, it\'s *directly* loaded, meaning
CFEngine variables are *not* expanded by default.

Say I have some JSON data file that contains CFEngine variables ( e.g.
`$(sys.workdir)`{.verbatim} ).

:::: captioned-content
::: caption
loading~structureddatathatcontainscfenginevariablesfromafile~.cf.json
:::

``` {.json tangle="loading_structured_data_that_contains_cfengine_variables_from_a_file.cf.json"}
{
  "policy_source": {
    "meta": "$(sys.workdir)/my_org/repo/test01_config/meta/",
    "file": "",
    "template": "",
    "module": "",
    "policy": "$(sys.workdir)/my_org/repo/test01_config/policy/"
  }
}
```
::::

I want to iterate over the keys of `policy_source`{.verbatim} and print
out the values that match the regular expression `.*meta.*`{.verbatim}
(e.g. `$(sys.workdir)/my_org/repo/test01_config/meta/`{.verbatim}).

In CFEngine, promises are usually skipped if they contain something that
looks like an unexpanded CFEngine variable. The thought is that we
don\'t want to start taking action when we have missing data. Perhaps
that data will be available in a subsequent pass. You can use the
`data_expand()` function to dereference CFEngine variables.

``` {.cfengine3 tangle="loading_structured_data_that_contains_cfengine_variables_from_a_file.cf"}
bundle agent __main__
{
  vars:

      "json_file"
        string => "loading_structured_data_that_contains_cfengine_variables_from_a_file.cf.json";


      "hosts_data" data => readjson("$(json_file)");

      # use data_expand() to resolve the cfengine variables inside the JSON file while loading the data.
      "expanded_hosts_data" data => data_expand( readjson("$(json_file)") );

      "l_data_types"
        slist => { "meta","policy","file" };

  reports:

      # If meta matches the iterator
      "hosts_data[policy_source][$(l_data_types)]: $(hosts_data[policy_source][$(l_data_types)])"
        if => regcmp(".*meta.*","$(hosts_data[policy_source][$(l_data_types)])");

      "expanded_hosts_data[policy_source][$(l_data_types)]: $(expanded_hosts_data[policy_source][$(l_data_types)])"
        if => regcmp(".*meta.*","$(expanded_hosts_data[policy_source][$(l_data_types)])");

}
```

Running the agent with `--log-level debug --log-=modules=all`{.verbatim}
(and redirecting to a file, since it will produce a LOT of output) we
can then see that the function was skipped for having arguments that
contain unresolved variables.

``` example
  debug: VariableTableGet(default:this.l_data_types): string  => meta
  debug: ExpandScalar( (null) : (null) . hosts_data[policy_source][$(l_data_types)] )  =>  hosts_data[policy_source][meta]
  debug: VariableTableGet(hosts_data[policy_source][meta]): NOT FOUND
  debug: VariableTableGet(hosts_data): NOT FOUND
  debug: VariableTableGet(default:this.hosts_data[policy_source][meta]): string  => $(sys.workdir)/my_org/repo/test01_config/meta/
  debug: ExpandScalar( (null) : (null) . $(hosts_data[policy_source][$(l_data_types)]) )  =>  $(sys.workdir)/my_org/repo/test01_config/meta/
  debug: Skipping function evaluation for now, arguments contain unresolved variables: regcmp(".*meta.*","$(hosts_data[policy_source][$(l_data_types)])")
  debug: Function call in class expression did not succeed
verbose: Skipping promise 'hosts_data[policy_source][$(l_data_types)]: $(hosts_data[policy_source][$(l_data_types)])' because constraint 'if => regcmp(".*meta.*","$(hosts_data[policy_source][$(l_data_types)])")' is not met
```
