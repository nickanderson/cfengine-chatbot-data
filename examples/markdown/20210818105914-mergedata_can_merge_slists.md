:::: captioned-content
::: caption
Mergedata can merge slists
:::

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both"}
bundle agent __main__
{
  vars:
      "l1" slist => { "one", "two" };
      "l2" slist => { "three", "four" };

      "m" data => mergedata( l1, l2 );

  reports:
      "$(m)";
}
```
::::

``` example
R: one
R: two
R: three
R: four
```

It can merge lists that are values of classic array keys:

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both"}
bundle agent __main__
{
  vars:
      "l1[one]" slist => { "one", "two" };
      "l2[two]" slist => { "three", "four" };

      "m" data => mergedata( "l1[one]", "l2[two]" );

  reports:
      "$(m)";
}
```

``` example
R: one
R: two
R: three
R: four
```

# References

- [CFEngine Examples](id:38277465-771a-4db4-983a-8dfd434b1aff)
- [mergedata()](id:176a208f-384f-4543-aaf4-599d03cbfa5d)
