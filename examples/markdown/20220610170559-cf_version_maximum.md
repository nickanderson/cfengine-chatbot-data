``` {.cfengine3 tangle="cf_version_maximum.cf"}
bundle agent __main__
{

  reports:
      "I am running $(sys.cf_version)";
      "This will be printed on versions of CFEngine up to 3.20.0."
        if => cf_version_maximum( "3.20.0" );

      "This will be printed on versions of CFEngine up to 3.18.0, not on 3.18.1 or later"
        if => cf_version_maximum( "3.18.0" );

}
```
