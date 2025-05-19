<https://rosettacode.org/wiki/Compare_length_of_two_strings>

Given two strings of different length, determine which string is longer
or shorter. Print both strings and their length, one on each line. Print
the longer one first.

Measure the length of your string in terms of bytes or characters, as
appropriate for your language. If your language doesn\'t have an
operator for measuring the length of a string, note it.

Extra credit

Given more than two strings: list =
\[\"abcd\",\"123456789\",\"abcdef\",\"1234567\"\] Show the strings in
descending length order.

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both" tangle="rosetta_code_compare_length_of_two_strings.cf"}
bundle agent __main__
{
  vars:
      "strings" slist => { "abcd", "123456789", "abcdef", "1234567" };

      "sorted[$(with)]"
        string => "$(strings)",
        with => string_length( "$(strings)" );

      "sort_idx" slist => reverse( sort( getindices( "sorted" ), lex ) );

  reports:
      "'$(sorted[$(sort_idx)])' is $(sort_idx) characters in length.";
}
```

``` example
R: '123456789' is 9 characters in length.
R: '1234567' is 7 characters in length.
R: 'abcdef' is 6 characters in length.
R: 'abcd' is 4 characters in length.
```

# References

- [CFEngine Examples](id:38277465-771a-4db4-983a-8dfd434b1aff)
- [reverse()](id:7f0a58dc-904b-45cf-bd30-c7c0c15b08ca)
- [string~length~()](id:d9397df0-ba02-45dc-a610-fea3d656b44f)
- [sort()](id:edde1a7e-c787-46b8-8ae1-285f12dafd90)
- [getindices()](id:cb299172-277a-42de-a1c9-c82e54379e4e)
