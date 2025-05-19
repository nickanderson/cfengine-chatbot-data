I want to build a map of hostnames to hostkeys based on the data
provided by `cf-key -s`.

You can use `parsestringarrayidx()` to parse a string into a data
structure (classic array). `cf-key` output to parse:

``` example
[root@hub ~]# cf-key -sN
Direction       IP      Name    Last connection Key
Incoming        192.168.33.4    -       Tue Jul 20 16:25:36 2021        SHA=341ca15d40ebb01936f2f8aa3298e89f1b7a2fb592c2401dc0de4403469c49fe
Outgoing        192.168.33.4    -       Tue Jul 20 16:30:55 2021        SHA=341ca15d40ebb01936f2f8aa3298e89f1b7a2fb592c2401dc0de4403469c49fe
Incoming        192.168.33.2    hub     Tue Jul 20 16:30:55 2021        SHA=503ef520e6fe7573246b38210b7d4e07b77eb77f6d0ff302eacbf67dac48cc4e
Outgoing        192.168.33.2    hub     Tue Jul 20 16:30:55 2021        SHA=503ef520e6fe7573246b38210b7d4e07b77eb77f6d0ff302eacbf67dac48cc4e
Incoming        192.168.33.3    host001 Tue Jul 20 16:26:42 2021        SHA=98c96717ac710a5567240ab82aec2bf4eaf8a35f91b41ad14f14e8be37cf26ab
Outgoing        192.168.33.3    host001 Tue Jul 20 16:30:55 2021        SHA=98c96717ac710a5567240ab82aec2bf4eaf8a35f91b41ad14f14e8be37cf26ab
Total Entries: 6
```

``` {.cfengine3 tangle="map_hostname_to_hostkey_by_parsing_cf_key_sn_output.cf"}
bundle agent __main__
{
  vars:
      "_results"
        int => parsestringarrayidx(
                                    _hostname_to_hostkey_map,                  # Classic array to hold parsed result
                                    execresult("$(sys.cf_key) -sN", noshell),  # input
                                    "^(Outgoing|Total|Direction)[^\n]*",       # comment
                                    "\s+",                                     # split
                                    inf,                                       # maxentries
                                    inf);                                      # maxbytes

      "_i" slist => getindices( _hostname_to_hostkey_map );

      "hostname_to_hostkey_map[$(_hostname_to_hostkey_map[$(_i)][2])]"
        string => "$(_hostname_to_hostkey_map[$(_i)][8])";

  reports:
      "$(with)" with => storejson( mergedata( hostname_to_hostkey_map ));
      "WARNING: Not all hosts have reverse lookups. Some hosts are missing from this map."
        if => some( "-", getindices( hostname_to_hostkey_map ) );

    DEBUG::
      "$(with)" with => storejson( mergedata( _hostname_to_hostkey_map ));
}
```

Execution Output:

``` example
[root@hub ~]# cf-agent -KIf ./t.cf
R: {
  "-": "SHA=341ca15d40ebb01936f2f8aa3298e89f1b7a2fb592c2401dc0de4403469c49fe",
  "host001": "SHA=98c96717ac710a5567240ab82aec2bf4eaf8a35f91b41ad14f14e8be37cf26ab",
  "hub": "SHA=503ef520e6fe7573246b38210b7d4e07b77eb77f6d0ff302eacbf67dac48cc4e"
}
R: WARNING: Not all hosts have reverse lookups. Some hosts are missing from this map.
```

**Note:** Hosts without reverse DNS entries will be problematic as
`-`{.verbatim} is used in place of a hostname.

In CFEngine Enterprise, there are other ways to get this information as
well, for example the `hostseen()` and `hostswithclass()` functions can
be executed on an enterprise hub and they can return a list of hostnames
or IP addresses from hosts that have reported to the Enterprise Hub.
Also, the *[hosts
API](https://docs.cfengine.com/latest/reference-enterprise-api-ref-host.html)*
and *[inventory
API](https://docs.cfengine.com/latest/reference-enterprise-api-ref-inventory.html)*
are also good sources for this information.
