``` {.cfengine3 tangle="execresult_as_data.cf"}
bundle agent __main__
{
  reports:
      "$(with)"
        with => storejson( execresult_as_data( "$(sys.bindir)/cf-agent --version", "noshell" ) );
}
```
