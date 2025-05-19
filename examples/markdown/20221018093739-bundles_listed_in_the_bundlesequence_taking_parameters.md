Can bundles listed in the bundlesequence take parameters?

Yes.

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both" tangle="bundles_listed_in_the_bundlesequence_taking_parameters.cf"}
body common control
{
   bundlesequence => {
                       mine( "my parameter value" )
   };
}
bundle agent mine( param )
{
  reports: "Hi got '$(param)'";
}
```

``` example
R: Hi got 'my parameter value'
```

Can you supply parameters to bundles via the
`--bundlesequence`{.verbatim} option?

No.
