:::: captioned-content
::: caption
exception~forpatchingpolicy~.cf
:::

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both" tangle="exception_for_patching_policy.cf"}
bundle agent __main__
{
  vars:
      "regular_patching_exception_flagfile" string => "/tmp/regular-patch.exception";

      # I like to encourage leaving tidbits of information that can be reported
      # on. E.g. you could even ignore the flag file if it's not populated with
      # any text.

      "regular_patching_exception_reason"
        string => readfile( "$(regular_patching_exception_flagfile)", 1k),
        meta => { "inventory",
                  "attribute_name=Regular Patching Exception Reason"},
        comment => concat( "We inventory the content of the file in case ",
                           "someone has blessed us with an explanation."),
        if => fileexists( $(regular_patching_exception_flagfile) );


    # I like to inventory when something started, or even dynamically calculate
    # the amount of time (minutes, or days) something has been in effect for
    # reporting. This can be very useful for building Dashboard Alerts and or
    # Compliance Report Checks. For example, you could have a check in a
    # compliance report that regular patching has not been in exception for more
    # than N days (use eval() and now() to calculate days since ctime and
    # inventory that, then the condition could be that this inventory item is not > N days).


      "regular_patching_exception_start"
        string => strftime( localtime,
                            "%A, %B %d %Y %Z", # This won't sort well, but is easy for humans to read
                                              # Simply re-format the time string to whatever works well for you
                            filestat( "$(regular_patching_exception_flagfile)", ctime )),
        meta => { "inventory",
                  "attribute_name=Regular Patching Exception Start"},
        comment => concat( "We inventory the creation time of the file as it's a good ",
                           "indication of when the exception started."),
        if => fileexists( $(regular_patching_exception_flagfile) );

      "regular_patching_exception_active_days"
        int => int( eval( "($(with)-$(sys.systime))/86400", math) ),
        meta => { "inventory",
                  "attribute_name=Regular Patching Exception Active Days"},
        with => filestat( "$(regular_patching_exception_flagfile)", ctime );

    files:

    # You might want to automatically clean up the flag file based on some
    # conditon, for example here if the file has been here for too many days

      "$(regular_patching_exception_flagfile)"
        delete => default:tidy,
        if => isgreaterthan( $(regular_patching_exception_active_days),
                             $(regular_patching_exception_max_days) ),
        comment => concat( "We only allow regular patching to be skipped ",
                           "for $(regular_patching_exception_max_days) days ",
                           "at which time the file is automatically removed." );
}
```
::::

:::: captioned-content
::: caption
Shell command with stderr
:::

``` {.bash org-language="sh" results="output" exports="both"}
exec 2>&1
echo "Because Nick said so" > /tmp/regular-patch.exception
chmod 600 ./exception_for_patching_policy.cf
cf-agent -Kf ./exception_for_patching_policy.cf --show-evaluated-vars=default:main\\.
:
```
::::

``` example
Variable name                            Variable value                                               Meta tags                                Comment
default:main.regular_patching_exception_active_days 0                                                            source=promise,inventory,attribute_name=Regular Patching Exception Active Days
default:main.regular_patching_exception_flagfile /tmp/regular-patch.exception                                 source=promise
default:main.regular_patching_exception_reason Because Nick said so                                         source=promise,inventory,attribute_name=Regular Patching Exception Reason We inventory the content of the file in case someone has blessed us with an explanation.
default:main.regular_patching_exception_start Friday, January 06 2023 CST                                  source=promise,inventory,attribute_name=Regular Patching Exception Start We inventory the creation time of the file as it's a good indication of when the exception started.
```

# Reporting in Mission Portal [[ATTACH]{.smallcaps}]{.tag tag-name="ATTACH"} {#dc9ee193-8716-4ca7-b8aa-57d7997605af}

Then in Mission Portal, these will be available in Inventory.

```{=org}
#+DOWNLOADED: file:///tmp/Spectacle.HhOUEJ/Screenshot_2023-01-06_142208.png @ 2023-01-06 14:22:11
```
![](attachment:2023-01-06_14-22-11_Screenshot_2023-01-06_142208.png)

And then you can also easily use the data in a compliance report:

```{=org}
#+DOWNLOADED: file:///tmp/Spectacle.HhOUEJ/Screenshot_2023-01-06_142140.png @ 2023-01-06 14:21:46
```
![](attachment:2023-01-06_14-21-46_Screenshot_2023-01-06_142140.png)
