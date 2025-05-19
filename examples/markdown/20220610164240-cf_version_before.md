``` {.cfengine3 tangle="cf_version_before.cf"}
bundle agent __main__
{

  reports:
      "The current version of CFEngine ($(sys.cf_version)) was not released before 3.18.0"
        unless => cf_version_before( "3.18.0" );

      "The current version of CFEngine ($(sys.cf_version)) was not released before 3.3"
        unless => cf_version_before( "3.3" );

      "The current version of CFEngine ($(sys.cf_version)) was released before 4.3.2"
        if => cf_version_before( "4.3.2" );
}
```
