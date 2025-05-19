Here are a few different ways to take
`"numbered" slist => {'1.txt','2.txt','3.txt'};` and get a list where
`.txt`{.verbatim} is stripped from each element.

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both" extra-opts="--show-evaluated-vars=default:main\\\\." tangle="a_few_ways_to_strip_suffixes_from_a_list.cf"}
bundle agent __main__
{

  vars:

      "numbered" slist => {'1.txt','2.txt','3.txt'};

      # Using maplist and basename()
      "l_stripped_0"
        slist => maplist( basename( "$(this)", ".txt" ), numbered );

      # Using an associative (aka classic) array, basename and getvalues()
      "a_stripped_1[$(numbered)]"
        string => basename( "$(numbered)", ".txt" );
      "l_stripped_1" slist => getvalues( "a_stripped_1" ); # no order promised!
      "l_stripped_1a" slist => sort( getvalues( "a_stripped_1" ), "int"  ); # sorted numerically

      # Using associatiative (aka classic) array, with regex replace, nth and with
      # to get the desirable portion of the string into both the key and the value

      "a_stripped_2[$(with)]"
        with => regex_replace( $(numbered), "(.*).txt", "$1", "" ),
        string => nth( string_split( $(numbered), ".txt", inf ), 0);
      "l_stripped_2a" slist => getindices( a_stripped_2 ); # no order promised!
      "l_stripped_2b" slist => getvalues( a_stripped_2 ); # no order promised!

      #By converting to a string, and back to a list while using regex_replace to munge the string
      "l_stripped_3"
        slist => string_split( regex_replace( join( " ", numbered ), ".txt", "", "g" ),
                               "\s+", inf ) ;
}
```

``` example
Variable name                            Variable value                                               Meta tags                                Comment
default:main.a_stripped_1[1.txt]         1                                                            source=promise
default:main.a_stripped_1[2.txt]         2                                                            source=promise
default:main.a_stripped_1[3.txt]         3                                                            source=promise
default:main.a_stripped_2[1]             1                                                            source=promise
default:main.a_stripped_2[2]             2                                                            source=promise
default:main.a_stripped_2[3]             3                                                            source=promise
default:main.l_stripped_0                 {"1","2","3"}                                               source=promise
default:main.l_stripped_1                 {"3","2","1"}                                               source=promise
default:main.l_stripped_1a                {"1","2","3"}                                               source=promise
default:main.l_stripped_2a                {"2","1","3"}                                               source=promise
default:main.l_stripped_2b                {"2","1","3"}                                               source=promise
default:main.l_stripped_3                 {"1","2","3"}                                               source=promise
default:main.numbered                     {"1.txt","2.txt","3.txt"}                                   source=promise
```
