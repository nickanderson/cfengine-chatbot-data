``` {.cfengine3 tangle="subshells_with_backticks_in_execresult.cf"}
bundle agent __main__
{
  vars:
      # We got this datetime string that we parsed from external data
      "kv[report_datetime_end]" string => "2022-04-03 12:10:19";

      # We calculate the number of days since the report finished.
      # execresult() is used to get the date in seconds since unix epoch 
      "report_date_days_since"
        string => format( "%d", eval( execresult( 'echo \(\($(sys.systime) - `date +%s --date "$(kv[report_datetime_end])"`\)/\(3600*24\)\)', useshell ), math, infix )),
        meta => { "inventory", "attribute_name=CISOfy Lynis days since scan completed"},
        if => isvariable( "kv[report_datetime_end]" );

  reports:
      "It's currently '$(sys.date)'";
      "It's been '$(report_date_days_since)' days since '$(kv[report_datetime_end])'";
}

```

Example Execution Results:

``` example
: R: It's currently 'Mon Apr  4 11:19:44 2022'
: R: It's been '0' days since '2022-04-03 12:10:19'
```

``` example
: R: It's currently 'Mon Apr  4 11:19:20 2022'
: R: It's been '1' days since '2022-04-03 11:10:19'
```

Alternatively, avoid subshell

``` {.cfengine3 tangle="subshells_with_backticks_in_execresult.cf"}
bundle agent __main__
{
  vars:
      # We got this datetime string that we parsed from external data
      "kv[report_datetime_end]" string => "2022-04-03 12:10:19";

      # We calculate the number of days since the report finished.
      # execresult() is used to get the date in seconds since unix epoch 
      "report_date_days_since"
        string => format( "%d", eval( '(($(sys.systime) - $(with))/(3600*24))', math, infix )),
        meta => { "inventory", "attribute_name=CISOfy Lynis days since scan completed"},
        if => isvariable( "kv[report_datetime_end]" ),
        with => execresult( "date +%s --date '$(kv[report_datetime_end])'", useshell);

  reports:
      "It's currently '$(sys.date)'";
      "It's been '$(report_date_days_since)' days since '$(kv[report_datetime_end])'";

}
```

# References

- [CFEngine Examples](id:38277465-771a-4db4-983a-8dfd434b1aff)
- [format()](id:5da3dcf8-5ea7-41e9-92a2-23e3755ad6fd)
- [isvariable()](id:02720c30-efe9-4bb8-b360-fbf79886a13d)
- [eval()](id:24647e3a-2af2-4460-897d-5b539bff2171)
- [execresult()](id:4f9b79dc-b8cc-4fcf-bb74-09792aa3d9eb)
