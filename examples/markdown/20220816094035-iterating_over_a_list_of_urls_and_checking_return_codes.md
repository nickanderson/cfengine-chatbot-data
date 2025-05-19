``` {.cfengine3 include-stdlib="t" log-level="info" exports="both" command-in-result="t" tangle="iterating_over_a_list_of_urls_and_checking_return_codes.cf"}
bundle agent __main__
{
  methods:
      "example_down_detector";
}
bundle agent example_down_detector
{
  vars:

      "urls"
        slist => { "cfengine.com", "notcfengine.com" };

     # Vars of type data can not be defined as arrays. So, for this case where we
     # are iterating we need to iterate on something that can be used in a
     # variable name, or we need to tag individual variables so that we can find
     # them later.
     # "urls_idx" slist => getindices( urls ); # Unfortunately, you can't simply pull the numeric positions from a list
     "urls_length" int => length( urls );
     "urls_idx"
      slist => expandrange( "[0-$(with)]", 1),
      with => eval( "$(urls_length)-1", math, infix);

     "url_get_opts" data => '{}';

    "ret_$(urls_idx)"
      data => url_get( nth( urls, $(urls_idx) ), @(url_get_opts) );


  reports:
      "$(urls_idx)";
      "Return Code: $(with) $(ret_$(urls_idx)[returncode])"
        with => nth( urls, $(urls_idx) );

 }
```

``` example
# cf-agent --no-lock --log-level info --file /home/nickanderson/org/roam/daily/work/cfengine3-oEbTei
R: 0
R: 1
R: Return Code: cfengine.com 301
R: Return Code: notcfengine.com 0
```
