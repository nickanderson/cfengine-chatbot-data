``` {.cfengine3 tangle="countlinesmatching.cf"}
bundle agent __main__
{
  vars:
      "reg" string => ".*agent.*";

  reports:
      "$(with) lines in this file matching '$(reg)'"
        with => countlinesmatching( $(reg), $(this.promise_filename) );
}
```
