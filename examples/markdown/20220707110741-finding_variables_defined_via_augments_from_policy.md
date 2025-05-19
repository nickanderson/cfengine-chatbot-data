[CFEngine Examples](id:38277465-771a-4db4-983a-8dfd434b1aff)

> How would I be able to list vars defined in augments in a policy?

You could read in Augments yourself, and use `getindices()`. But I think
there are various reasons that I would send this to the bottom of my
list. I think it would be better to search based on metadata like the
`source=augments_file` tag.

Consider this example `def.json`{.verbatim}.

`/tmp/6001/def.json`{.verbatim}:

:::: captioned-content
::: caption
/tmp/6001/def.json
:::

``` {.json tangle="/tmp/6001/def.json"}
{
    "variables":
    {
        "MyVar": {
            "value": "MyVal"
        },
        "MyBundle.MyVars": {
            "value": [ "My", "Vals" ]
        },
        "MyNamespace:YourBundle.MyData": {
            "value": {
                "has": "more",
                "stuff": [ "deep", "inside" ]
            }
        }
    },
    "vars":
    {
        "MyVar": "MyVal",
        "MyVars": [ "My", "Vals" ],
        "MyData": {
            "has": "more",
            "stuff": [ "deep", "inside" ]
        }
    }
}
```
::::

`/tmp/6001/vars-from-augments.cf`{.verbatim}:

:::: captioned-content
::: caption
/tmp/6001/vars-from-augments.cf
:::

``` {.cfengine3 tangle="finding_variables_defined_via_augments_from_policy.cf"}
bundle agent __main__
{
   vars:
    "variablesmatching" slist => variablesmatching( ".*", "source=augments_file" );
    "variablesmatching_as_data" data => variablesmatching_as_data( ".*", "source=augments_file" );


  reports:
    'variablesmatching( ".*", "source=augments_file" ) found $(variablesmatching)';
    'variablesmatching_as_data( ".*", "source=augments_file" ) found $(with)'
      with => storejson( variablesmatching_as_data );
}
```
::::

Running the policy:

:::: captioned-content
::: caption
Running the policy
:::

``` {.bash org-language="sh" results="output" exports="both"}
exec 2>&1
chmod 600 /tmp/6001/vars-from-augments.cf
cf-agent -Kf /tmp/6001/vars-from-augments.cf
:
```
::::

Execution Output:

``` example
R: variablesmatching( ".*", "source=augments_file" ) found default:def.MyData
R: variablesmatching( ".*", "source=augments_file" ) found default:def.MyVars
R: variablesmatching( ".*", "source=augments_file" ) found default:MyBundle.MyVars
R: variablesmatching( ".*", "source=augments_file" ) found default:def.MyVar
R: variablesmatching( ".*", "source=augments_file" ) found MyNamespace:YourBundle.MyData
R: variablesmatching_as_data( ".*", "source=augments_file" ) found {
  "MyNamespace:YourBundle.MyData": {
    "has": "more",
    "stuff": [
      "deep",
      "inside"
    ]
  },
  "default:MyBundle.MyVars": [
    "My",
    "Vals"
  ],
  "default:def.MyData": {
    "has": "more",
    "stuff": [
      "deep",
      "inside"
    ]
  },
  "default:def.MyVar": "MyVal",
  "default:def.MyVars": [
    "My",
    "Vals"
  ]
}
```

Note: The example shows the use of the *variables* key which was added
in 3.18.0 along with other Augments extensions like
`host_specific.json`{.verbatim}.
