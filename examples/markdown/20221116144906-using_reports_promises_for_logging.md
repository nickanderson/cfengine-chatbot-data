:::: captioned-content
::: caption
Example Policy
:::

``` {.cfengine3 include-stdlib="nil" log-level="info" exports="both" tangle="using_reports_promises_for_logging.cf"}
bundle agent __main__
{

  reports:
      "$(sys.date) I changed the MOTD"
        report_to_file => "/tmp/cfengine.log";

      "Let's see it"
        printfile => cat( "/tmp/cfengine.log" );

}
```
::::

``` example
R: Let's see it
R: Wed Nov 16 13:40:22 2022 I changed the MOTD
```
