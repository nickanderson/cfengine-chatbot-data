In CFEngine promises are executed in the *Normal Order*.

For example:

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both"}
bundle agent main
{
  reports:
    "First";
    "Second";
    "Third";
}
```

``` example
R: First
R: Second
R: Third
```

The depends~on~ attribute can be used to delay a promise actuation until
one or more promises (identified by their handle) are seen to be *kept*
or *repaired*.

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both"}
bundle agent main
{
  reports:
    "First" handle => "first", depends_on => { "second" };
    "Second" handle => "second", depends_on => { "third" };
    "Third" handle => "third";
}
```

``` example
R: Third
R: Second
R: First
```

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both"}
bundle agent main
{
  reports:
    "First" handle => "first", depends_on => { "second" };
    "Second" handle => "second", depends_on => { "third" };
    "Third" handle => "third";
}
```

Note that it\'s possible to create a condition that will not be
resolved.

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both"}
bundle agent main
{
  reports:
    "Classic chicken and egg problem";
    "I have a chicken" handle => "chicken", depends_on => { "egg" };
    "I have an Egg" handle => "egg", depends_on => { "chicken" };
}
```

``` example
R: Classic chicken and egg problem
```

Note that *all* dependancies must be considered *kept* or *repaired*
before the promise will actuate.

It\'s possible to implement similar behavior with classes. For example,
the *results* classes body from the stdlib defines classes for *all*
promise outcomes.
