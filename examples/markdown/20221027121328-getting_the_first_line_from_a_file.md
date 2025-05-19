``` {.cfengine3 tangle="getting_the_first_line_from_a_file.cf"}
bundle agent __main__
{
  vars:
    "i" string => nth( readstringlist( $(this.promise_filename), "", "$(const.n)", 1, inf ), 0 );

  reports:
    "$(i)";
}

```
