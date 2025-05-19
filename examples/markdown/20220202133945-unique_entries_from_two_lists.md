:::: captioned-content
::: caption
Unique entries from two lists
:::

``` {.cfengine3 tangle="unique_entries_from_two_lists.cf"}
bundle agent __main__
{
      vars:
      "a" slist => { "1", "2", "3" };
      "b" slist => { "3", "4", "5" };
      "ab" slist => { @(a), @(b) };
      "u" slist => unique( @(ab) );
 reports:
      "$(u)";
}
```
::::

**References:**

- [CFEngine Examples](id:38277465-771a-4db4-983a-8dfd434b1aff)
- [function-unique()](id:11d6f564-8a74-4041-9635-c24b72e065f8)
