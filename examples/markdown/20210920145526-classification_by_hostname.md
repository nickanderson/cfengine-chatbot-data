Classification based on hostname is a common pattern. Use a regular
expression to classify a host based on a host name convention.

:::: captioned-content
::: caption
Example Policy
:::

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both"}
bundle agent __main__
{
  vars:
      "_hostname" string => "prd-aue1-b-c7-p-distrib-001";

      "_hostname_parts"
        data => data_regextract("^(?<environment>\w+)-(?<thing2>\w+)-(?<thing3>\w+)-(?<thing4>\w+)-(?<thing5>\w+)-(?<thing6>\w+)-(?<number>\d+)$", $(_hostname));

  classes:
      "environment_$(_hostname_parts[environment])"
        expression => "any",
        comment => "Having a class for the environment part of the hostname let's target different environmnets";

  reports:

      "Parsed $(const.dollar)(_hostname): $(with)"
        with => storejson( _hostname_parts );

    environment_prd::
      "I am in the prd environment";
}
```
::::

``` example
R: Parsed $(_hostname): {
  "0": "prd-aue1-b-c7-p-distrib-001",
  "environment": "prd",
  "number": "001",
  "thing2": "aue1",
  "thing3": "b",
  "thing4": "c7",
  "thing5": "p",
  "thing6": "distrib"
}
R: I am in the prd environment
```

While this is a common pattern, it\'s not one that I recommend.

Advantages:

- Simple
- Common/Familiar

Disadvantages:

- Hostname convention 1-N
  - Typically someone identifies new information that would be useful to
    classify by and they re-engineer the hostname scheme to account for
    the new metadata, commonly invalidating previous naming conventions.
- Limited space to encode information
- Easily fungible with root access on a host, simply change the host
  name.

# References

- [CFEngine Examples](id:38277465-771a-4db4-983a-8dfd434b1aff)
- Function [data~regextract~()](id:4e5937cb-3de1-4e7e-9756-9a4f469fe4bb)
- [Function: storejson](id:dec0586a-18c6-4dd3-bf22-78a00f9ddb3f)
