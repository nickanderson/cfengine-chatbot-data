``` {.cfengine3 tangle="classesmatching.cf"}
bundle agent __main__
{
  reports:
      "$(with)"
        with => join( ", ", classesmatching( "cfengine.*" ) );
}
```
