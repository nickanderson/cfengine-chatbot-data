Get N first lines from a file:

``` {.cfengine3 tangle="getting_first_n_lines_from_a_file.cf" extra-opts="--show-evaluated-vars=main\\\\."}
bundle agent __main__
{
  vars:
    "n" int => "2";
    "lines" slist => expandrange( "[0-$(with)]", 1 ), with => format( "%d", eval( "$(n)-1" ) );
    "line[$(lines)]" string => nth( readstringlist( $(this.promise_filename), "", "$(const.n)", $(n), inf ), $(lines) );
}
```

Using `int()` instead of `format()`.

``` {.cfengine3 tangle="getting_first_n_lines_from_a_file-int-instead-of-format.cf" extra-opts="--show-evaluated-vars=main\\\\."}
bundle agent __main__
{
  vars:
    "n" int => "2";
    "lines" slist => expandrange( "[0-$(with)]", 1 ), with => int( eval( "$(n)-1" ) );
    "line[$(lines)]" string => nth( readstringlist( $(this.promise_filename), "", "$(const.n)", $(n), inf ), $(lines) );
}
```
