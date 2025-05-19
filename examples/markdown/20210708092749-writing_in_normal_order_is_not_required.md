[CFEngine Examples](id:38277465-771a-4db4-983a-8dfd434b1aff)

:::: captioned-content
::: caption
Example illustrating that writing in the normal order is not required
:::

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both" tangle="writing_in_normal_order_is_not_required.cf"}
bundle agent main
{
  reports:
      "The message is '$(msg)'"
        handle => "MARK1";

  vars:
      "msg"
        string => "Hello",
        handle => "MARK2";
}
```
::::

``` example
R: The message is 'Hello'
```

Verbose output helps see the specific details.

:::: captioned-content
::: caption
Shell command with stderr
:::

``` {.bash org-language="sh" results="output" exports="both"}
exec 2>&1
cf-agent -Kvf ./writing_in_normal_order_is_not_required.cf | grep -P "BEGIN"
:
```
::::

``` example
 verbose: BEGIN parsing file: ./writing_in_normal_order_is_not_required.cf
 verbose: BEGIN Discovered hard classes:
 verbose: B: BEGIN bundle main
 verbose: V: BEGIN variables (pass 1)
 verbose: P: BEGIN promise 'MARK1' of type "reports" (pass 1)
 verbose: V: BEGIN variables (pass 2)
 verbose: V: BEGIN variables (pass 3)
```

I think I should see
\"`verbose: V: BEGIN promise 'MARK2' of type "vars" (pass 1)`{.verbatim}\",
but alas, I do not (even if I pass `--log-modules=all`.

So I filed a ticket, CFE-3706.
