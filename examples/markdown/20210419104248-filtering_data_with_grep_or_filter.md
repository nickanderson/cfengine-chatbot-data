```{=org}
#+roam_tags: CFEngine-example
```
I want to remove all the ones that does not include the entire path
within testdir

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both" extra-opts="--show-evaluated-vars=default:main\\\\."}
bundle agent __main__
{
  vars:
      "testdir" string => "/tmp";
      "_search_up_result" data => '[
 "$(testdir)/a/b/c/d/e/.",
 "$(testdir)/a/b/c/d/.",
 "$(testdir)/a/b/c/.",
 "$(testdir)/a/b/.",
 "$(testdir)/a/.",
 "$(testdir)/."
 "NOPE"
 ]';


      "filtered_with_filter"
        slist => filter( "^$(testdir)/.*",
                         _search_up_result, # list to filter
                         true, # regex
                         false, # Invert
                         inf); # max

      "filtered_with_grep"
        slist => grep( "^$(testdir)/.*", _search_up_result );

  reports:
      "search_up returned: $(_search_up_result)";
      "Filtered with filter(): $(filtered_with_filter)";
      "Filtered with grep(): $(filtered_with_grep)";

}
```

``` example
R: search_up returned: /tmp/a/b/c/d/e/.
R: search_up returned: /tmp/a/b/c/d/.
R: search_up returned: /tmp/a/b/c/.
R: search_up returned: /tmp/a/b/.
R: search_up returned: /tmp/a/.
R: search_up returned: /tmp/.
R: search_up returned: NOPE
R: Filtered with filter(): /tmp/a/b/c/d/e/.
R: Filtered with filter(): /tmp/a/b/c/d/.
R: Filtered with filter(): /tmp/a/b/c/.
R: Filtered with filter(): /tmp/a/b/.
R: Filtered with filter(): /tmp/a/.
R: Filtered with filter(): /tmp/.
R: Filtered with grep(): /tmp/a/b/c/d/e/.
R: Filtered with grep(): /tmp/a/b/c/d/.
R: Filtered with grep(): /tmp/a/b/c/.
R: Filtered with grep(): /tmp/a/b/.
R: Filtered with grep(): /tmp/a/.
R: Filtered with grep(): /tmp/.
Variable name                            Variable value                                               Meta tags                               
default:main._search_up_result           ["/tmp/a/b/c/d/e/.","/tmp/a/b/c/d/.","/tmp/a/b/c/.","/tmp/a/b/.","/tmp/a/.","/tmp/.","NOPE"] source=promise                          
default:main.filtered_with_filter         {"/tmp/a/b/c/d/e/.","/tmp/a/b/c/d/.","/tmp/a/b/c/.","/tmp/a/b/.","/tmp/a/.","/tmp/."} source=promise                          
default:main.filtered_with_grep           {"/tmp/a/b/c/d/e/.","/tmp/a/b/c/d/.","/tmp/a/b/c/.","/tmp/a/b/.","/tmp/a/.","/tmp/."} source=promise                          
default:main.testdir                     /tmp                                                         source=promise                          
```

# Reference {#1cb4fa82-4803-453d-8a34-ebfc33dda0eb}

- [Function: grep()](id:4480eb43-5bd1-4b76-bb9f-60216fc55795)
- [Function: filter()](id:72600c06-e3a9-4d48-b9ad-8756abe24442)
