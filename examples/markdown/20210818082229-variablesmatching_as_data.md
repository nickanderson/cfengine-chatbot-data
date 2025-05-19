`variablesmatching_as_data()` is useful for centralizing information
from disparate sources. For example, deriving a list of package names by
extracting the values from variables that match a specific name.

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both"}
 bundle agent package_names_from_variablesmatching_as_data
 # @brief Example illustrating deriving a list of package names from variables named with a specific naming convention
{
  vars:
      "packages_baseline" slist => { "bash-completion", "emacs" };
      "packages_os" slist => { "httpd" };
      "packages_barf" slist => { "apache" };

      "_mypkgdata" data => variablesmatching_as_data( ".*\.packages_.*" );

      "_v" slist => getvalues( _mypkgdata );

  reports:
      '$(with)'
        with => storejson( _v );

}
bundle agent __main__
{
      methods: "finding_vars_by_name";
}
```

``` example
R: [
  "bash-completion",
  "emacs",
  "httpd",
  "apache"
]
```

Or, slightly better, a tag:

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both"}
bundle agent package_names_from_variablesmatching_as_data
# @brief Example illustrating deriving a list of package names from variables having a specific tag
{
  vars:
      "packages_baseline"
        meta => { "desired_package_names=present" },
        slist => { "bash-completion", "emacs" };

      "packages_os"
        meta => { "desired_package_names=present" },
        slist => { "httpd" };

      "packages_barf"
        meta => { "desired_package_names=present" },
        slist => { "apache" };

      "_mypkgdata" data => variablesmatching_as_data( ".*", "desired_package_names=present" );

      "_v" slist => getvalues( _mypkgdata );

  reports:
      '$(with)'
        with => storejson( _v );

}
bundle agent __main__
{
      methods: "finding_vars_by_name";
}
```

``` example
R: [
  "bash-completion",
  "emacs",
  "httpd",
  "apache"
]
```

# References

- [CFEngine Examples](id:38277465-771a-4db4-983a-8dfd434b1aff)
- [variablesmatching~asdata~()](id:e1d06fa2-da14-4861-b724-0a0e46c99977)
- [getvalues()](id:d202c34d-21c3-46e9-a668-79fbdb61b9e7)
- [storejson()](id:a4b316dc-e357-4292-a43e-3cac1a55b50c)
