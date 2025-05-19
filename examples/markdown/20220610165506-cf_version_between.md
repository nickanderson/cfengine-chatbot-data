``` {.cfengine3 tangle="cf_version_between.cf"}
bundle agent __main__
{

  vars:
      "minor_version_next"
        string => format( "%d", eval( "$(sys.cf_version_minor)+1", math ) );

      "minor_version_previous"
        string => format( "%d", eval( "$(sys.cf_version_minor)-1", math ) );

  reports:
      "The current version of CFEngine ($(sys.cf_version)) is considered between 3.$(minor_version_previous).0 and 3.$(minor_version_next).0"
        if => cf_version_between( "3.$(minor_version_previous).0", "3.$(minor_version_next).0");
}
```
