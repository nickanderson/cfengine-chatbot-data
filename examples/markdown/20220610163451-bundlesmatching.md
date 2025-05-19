``` {.cfengine3 include-stdlib="nil" tangle="bundlesmatching.cf"}
bundle agent __main__
{

  reports:
      "bundlesmatching( '.*' ): $(with)"
        with => join( ", ", bundlesmatching( '.*' ) );

      "bundlesmatching( '.*', 'my-tag' ): $(with)"
        with => join( ", ", bundlesmatching( '.*', 'my-tag' ) );


}
bundle agent my_bundle
{
}
bundle agent your_bundle
{
  meta:
      "tags" slist => { "my-tag" };
}      
```

**References:**

- [CFEngine Examples](id:38277465-771a-4db4-983a-8dfd434b1aff)
- [bundlesmatching()](id:be2454de-2936-4d1a-b350-47ae1eb01a92)
