- Beware using variables in a methods type promisor, at least when
  combined with `callstack_callers()` the variable seems to be
  de-referenced when callstack~callers~() is called, not when the
  methods promise is actuated.

:::: captioned-content
::: caption
See what callstack~callers~() returns
:::

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both" extra-opts="--show-evaluated-vars=more" tangle="example_illustrating_callstack_promisers.cf"}
bundle agent __main__
{
   methods:
      # The zeroth promise, surprisingly to me, this.budnle is expanded not here, but where callstack_promisers is used!
      "Calling a method from main '$(this.bundle)'" -> { "Nick" }
        usebundle => play_with_callstack_promisers;
 }
bundle agent play_with_callstack_promisers
{
  methods:
      # The 1st promise
      "Please sir can i haz some more? '$(this.bundle)'" -> {"Anderson"}
        usebundle => more;
}

bundle agent more
{
  vars:
      "MSG" string => "Yummy";
      "promisers"
        slist => callstack_promisers();

  reports:
      "$(with)"
        with => nth( promisers, 0 );

      "$(with)"
        with => nth( promisers, 1 );

}
```
::::

e.g. I expected to see `Calling a method from main 'main'`{.verbatim}
instead of `Calling a method from main 'more'`{.verbatim}.

```{=org}
#+RESULTS:
```
``` example
R: Calling a method from main 'more'
R: Please sir can i haz some more? 'more'
Variable name                            Variable value                                               Meta tags                                Comment                                 
default:more.MSG                         Yummy                                                        source=promise                                                                   
default:more.promisers                    {"Calling a method from main 'more'","Please sir can i haz some more? 'more'"} source=promise                                                                   
```

**References:**

- [CFEngine Examples](id:38277465-771a-4db4-983a-8dfd434b1aff)

- [callstack~promisers~()](id:ce45d69c-1b0c-4b49-bc58-57b4c6d3c4ee)

- [nth()](id:912207eb-c307-4a33-bda5-18bf8a6410fc)

  ---

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both" extra-opts="--show-evaluated-vars=more" tangle="example_illustrating_callstack_promisers.cf"}
bundle agent __main__
{
  vars:
      "me" string => "__main__";

   methods:
      # The zeroth promise, surprisingly to me, variables in the promiser seem to be expanded not here, but where callstack_promisers is used!
      #"Calling a method from (__main__ aka main) this.bundle='$(this.bundle)'" -> { "Nick" }
      #"Calling a method from (__main__ aka main) me=$(me) this.bundle='$(this.bundle)'" -> { "Nick" }
      "Calling a method from (__main__ aka main) me=$(me) main.me=$(main.me) this.bundle='$(this.bundle)'" -> { "Nick" }
        usebundle => play_with_callstack_promisers;
 }
bundle agent play_with_callstack_promisers
{
  methods:
      # The 1st promise
      "Please sir can i haz some more? '$(this.bundle)'" -> {"Anderson"}
        usebundle => more;
}

bundle agent more
{
  vars:
      "MSG" string => "Yummy";
      "callstack_promisers"
        slist => callstack_promisers();

  reports:
      "$(with)"
        with => nth( callstack_promisers, 0 );

      "$(with)"
        with => nth( callstack_promisers, 1 );

}
```

``` example
R: Please sir can i haz some more? 'more'
R: Calling a method from (__main__ aka main) me=$(me) main.me=__main__ this.bundle='more'
Variable name                            Variable value                                               Meta tags                                Comment                                 
default:more.MSG                         Yummy                                                        source=promise                                                                   
default:more.callstack_promisers          {"Calling a method from (__main__ aka main) me=$(me) main.me=__main__ this.bundle='more'","Please sir can i haz some more? 'more'"} source=promise                                                                   
```
