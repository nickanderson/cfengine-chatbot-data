``` {.cfengine3 include-stdlib="t" verbose-mode="nil" inform-mode="nil" exports="both"}
bundle agent example
{
   reports:
     "CFEngine $(sys.cf_version) $(with)tab" with => format( $(const.t) );
      "CFEngine $(sys.cf_version) $(with)newline" with => format( $(const.n) );
}
bundle agent __main__
{
  methods:
      "example";
}
```

``` example
R: CFEngine 3.14.0a.ed0158a8e     tab
R: CFEngine 3.14.0a.ed0158a8e 
newline
```
