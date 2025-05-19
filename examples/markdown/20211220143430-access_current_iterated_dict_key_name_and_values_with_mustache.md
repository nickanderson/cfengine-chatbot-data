- `@`{.verbatim} Renders the currently iterated key
- `.`{.verbatim} Accesses a value

So, if you are iterating over the dict aliases which is a dict of dicts:

``` json
{
    "aliases": {
        "key1": {
            "msg": "Hello from 1",
            "msg2": "Hello2 from 1"
        },
        "key2": {
            "msg": "Hello from 2",
            "msg2": "Hello2 from 2"
        }
    }
}
```

You could do this:

:::: captioned-content
::: caption
Example Policy
:::

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both"}
bundle agent __main__
{
  vars:
      "d" data => '{
  "aliases": {
    "key1": {
      "msg": "Hello from 1", 
      "msg2": "Hello2 from 1" 
    },
    "key2": {
      "msg": "Hello from 2", 
      "msg2": "Hello2 from 2" 
    }
  }
}';

  reports:
       "$(with)"
         with => string_mustache( "$(const.n){{#aliases}} {{@}}:$(const.t){{.msg}}|{{{.msg2}}}$(const.n){{/aliases}}", d );


}
```
::::

``` example
R: 
 key1:    Hello from 1|Hello2 from 1
 key2:    Hello from 2|Hello2 from 2

```

:::: captioned-content
::: caption
Example Policy
:::

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both"}
  bundle agent __main__
  {

  vars:
      "d" data => '{
  "aliases": {
    "alice.alyson": "alice",
    "bob.bobberts": "bob",
    "carol.carolton": "carol"
  }
}';

  reports:
      "Aliases:$(const.n)$(with)"
        with => string_mustache( "{{#aliases}} {{@}}:$(const.t){{.}}$(const.n){{/aliases}}", d );
}
```
::::

``` example
R: Aliases:
 alice.alyson:    alice
 bob.bobberts:    bob
 carol.carolton:  carol

```

**References:**

- [CFEngine Examples](id:38277465-771a-4db4-983a-8dfd434b1aff)
- [string~mustache~()](id:7b2ad788-6a8d-4f1a-94e5-6947fd7e2431)
- [with =\>](id:5f995365-46bb-4701-80d8-12c163fc6ca8)
