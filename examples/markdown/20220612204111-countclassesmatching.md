``` {.cfengine3 tangle="countclassesmatching.cf"}
bundle agent __main__
{
      reports:
      "$(with) classes matching 'cfengine.*'"
        with => countclassesmatching( "cfengine.*" );
      "$(with) match 'cfengine.*"
        with => join( ", ", classesmatching( "cfengine.*" ) );
}
```
