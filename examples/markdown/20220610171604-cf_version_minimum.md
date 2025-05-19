``` {.cfengine3 tangle="cf_version_minimum.cf"}
bundle agent __main__
{
  reports:
      "I am running $(sys.cf_version)";

      "This message will start appearing on hosts running cfengine 3"
        if => cf_version_minimum( "3" );

      "This message will start appearing on hosts running cfengine 3.18"
        if => cf_version_minimum( "3.18" );

      "This message will start appearing on hosts running cfengine 3.18.1"
        if => cf_version_minimum( "3.18.1" );

      "This will stop appearing once a host is running 4.18.1"
        unless => cf_version_minimum( "4.18.1" );
}
```
