Note, `def.mpf_extra_autorun_inputs`{.verbatim} can be a *list* of
directories that contain `.cf`{.verbatim} suffixed files.

From the snippet you posted, note
`"_extra_autorun_inputs[$(_extra_autorun_input_dirs)]"`{.verbatim} is an
associative array. It has a key defined for each directory in
`def.mpf_extra_autorun_inputs`{.verbatim} with a value being a list of
`.cf`{.verbatim} suffixed files that reside within that directory.

However, this does not descend to find policy within sub-directories of
`def.mpf_extra_autorun_inputs`{.verbatim}.

For this type of extension, I think you are on the right track so far as
building your own autorun.

> Can I replace the services~autorun~ by custom one?

You *can* replace it by modifying the vendored policy, but I would
advise against it. Rather than *replacing* the vendored
`services_autoun`{.verbatim} you should be able to implement your own
autorun as an addition.

> would be this the way to do it?
> <https://docs.cfengine.com/docs/3.19/reference-components-file_control_promises.html#inputs>

Yes, I think using `inputs`{.verbatim} in `body file control`{.verbatim}
is probably the direction I would go.

Consider:

- Using the `inputs`{.verbatim} key in Augments to specify the path to
  the policy file which will contain the bundles to find and load the
  policy files you want.
- Using `def.control_common_bundlesequence_classification`{.verbatim} to
  specify the bundles that would find and load your policies as well as
  the bundle to run the found policies.

# Mocked Implementation

## The Layout

``` example
[root@hub MyPolicies]# find /var/cfengine/masterfiles/services/MyPolicies/
/var/cfengine/masterfiles/services/MyPolicies/
/var/cfengine/masterfiles/services/MyPolicies/one
/var/cfengine/masterfiles/services/MyPolicies/one/one.cf
/var/cfengine/masterfiles/services/MyPolicies/two
/var/cfengine/masterfiles/services/MyPolicies/two/sub1
/var/cfengine/masterfiles/services/MyPolicies/two/sub1/three.cf
/var/cfengine/masterfiles/services/MyPolicies/two/two.cf
/var/cfengine/masterfiles/services/MyPolicies/find-n-load.cf
/var/cfengine/masterfiles/services/MyPolicies/README.org
```

## Integration via Augments (def.json)

``` json
[root@hub MyPolicies]# python -m json.tool < /var/cfengine/masterfiles/def.json 
{
    "inputs": [
        "services/MyPolicies/find-n-load.cf"
    ],
    "vars": {
        "control_common_bundlesequence_classification": [
            "find_n_load_policies",
            "run_found_policies"
        ]
    }
}
```

## The Meat

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both"}
bundle common find_n_load_policies
# Find policy files
{
  vars:

      # Only cf-agent should find these policy files, we don't need cf-serverd,
      # cf-monitord, cf-execd spending time parsing these. We set an empty list
      # because the components won't like trying to load an unresolved variable.
      # Here we use the 'agent' class which is always defined when cf-agent is
      # the component executing the policy.

    agent::
      "search_root" string => "$(this.promise_dirname)/*/**.cf";
      "found_policy_files" slist => findfiles( $(search_root) );

    !agent::
      # non cf-agent components find an empty list.
      "found_policy_files" slist => {};

  reports:
    agent::
      'findfiles($(search_root)):';
      '$(found_policy_files)';
}
bundle agent run_found_policies
# Run found policy files
{
  vars:
      "b" slist => bundlesmatching( ".*", "MyTag" );

  methods:
      "$(b)";

  reports:
      "Found $(b) with tag 'MyTag'";
}

body file control
# Include the found inputs in the policy
{
        inputs => { @(find_n_load_policies.found_policy_files) };
}

bundle agent __main__
# Run these bundles if this file is run as the entry point.
{
  methods:
      "find_n_load_policies";
      "run_found_policies";
}
```

## Example Policy in subdir

Each of the example policies used are identical except for the bundle
name.

``` example
[root@hub MyPolicies]# cat /var/cfengine/masterfiles/services/MyPolicies/two/sub1/three.cf
bundle agent three
{
  meta:
    "tags" slist => { "MyTag" };

  reports:
    "Hello from $(this.bundle) in $(this.promise_filename)";
}
```

## Example execution output

``` example
[root@hub MyPolicies]# cf-agent -K
R: findfiles(/var/cfengine/inputs/services/MyPolicies/*/**.cf):
R: /var/cfengine/inputs/services/MyPolicies/one/one.cf
R: /var/cfengine/inputs/services/MyPolicies/two/two.cf
R: /var/cfengine/inputs/services/MyPolicies/two/sub1/three.cf
R: Hello from three in /var/cfengine/inputs/services/MyPolicies/two/sub1/three.cf
R: Hello from two in /var/cfengine/inputs/services/MyPolicies/two/two.cf
R: Hello from one in /var/cfengine/inputs/services/MyPolicies/one/one.cf
R: Found default:three with tag 'MyTag'
R: Found default:two with tag 'MyTag'
R: Found default:one with tag 'MyTag'
```
