``` {.cfengine3 tangle="getclassmetatags.cf"}
bundle agent __main__
{
  reports:
      "$(with)" with => join( ", ", getclassmetatags( "cfengine" ) );
}
```
