``` {.cfengine3 tangle="how_to_get_a_subset_of_n_elements_from_a_list.cf" extra-opts="--show-evaluated-vars=main\\\\." command-in-result="t"}
bundle agent __main__
{
  vars:
    "found" slist => { "too", "many", "things", "found" };
    "found_l" int => length( found );
    "limit" int => "2";
    "REAL_limit" string => format( "%d", eval( "$(limit)-1", math, infix) );
    "a" slist => expandrange( "[0-$(REAL_limit)]", 1 );
    "n[$(a)]"
      string => nth(found, $(a) );
    "limited" slist => getvalues( n );

}
```
