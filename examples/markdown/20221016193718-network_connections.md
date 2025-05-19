``` {.cfengine3 tangle="network_connections.cf"}
bundle agent __main__
{
  reports:
      "$(with)" with => storejson( network_connections());
}
```
