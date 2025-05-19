``` {.cfengine3 include-stdlib="t" log-level="info" exports="both" wrap="example" tangle="host_specific_cmdb_data_files.cf"}
bundle agent __main__
{
  vars:

      "_d" data => '{
    "data": [
        {
            "id": "SHA=50d3ca3f9f0bcf02e2386ff86240bbc0d63e09764db052f97fec92eb84ecf449",
            "vars": {
                "Namespace:BundleName.VariableName": { "value": "myvalue",
                                                       "meta": {
                                                           "derived_from": "host_specific.json (inserted by agent automatically)",
                                                           "comment": "def.json will not overwrite variables set by cmdb data, this applies to data files by augments key",
                                                           "description": "so much meta"
                                                       }
                                                     },

                "HubCMDB:My.hostname": { "value": "host1.cfengine.com",
                                         "meta": {
                                             "comment": "My hostname should be set to this",
                                             "description": "The name of the host" }
                                       },

                "HubCMDB:My.pypy_password": { "secret_file": "secret1.cfsecret",
                                              "secret": "*($@*&%(&$#@))",
                                              "meta": {
                                                  "comments": ["inline secret advantage being simple",
                                                               "disadvantage: could dramatically increase volume of data unnecessarily transmitted to each Agent ",
                                                               "secretes encrypted for single host best inline",
                                                               "secretes encrypted for multiple hosts best in separate file"
                                                              ],
                                                  "description": "The name of the host" }
                                            }
            },

            "classes": {
                "My_class": {
                    "meta": { "comment": "i love this one", "description": "so much meta" }
                }
            }
        },
        {
            "id": "SHA=40d3ca3f9f0bcf02e2386ff86240bbc0d63e09764db052f97fec92eb84ecf449",
            "vars": {
                "Namespace:BundleName.VariableName": { "value": "myvalue",
                                                       "meta": {
                                                           "derived_from": "host_specific.json (inserted by agent automatically)",
                                                           "comment": "def.json will not overwrite variables set by cmdb data, this applies to data files by augments key",
                                                           "description": "so much meta"
                                                       }
                                                     },

                "HubCMDB:My.hostname": { "value": "host2.cfengine.com",
                                         "meta": {
                                             "comment": "My hostname should be set to this",
                                             "description": "The name of the host" }
                                       },

                "HubCMDB:My.pypy_password": { "secret_file": "secret2.cfsecret",
                                              "secret": "*($@*&%(&$#@))",
                                              "meta": {
                                                  "comments": ["inline secret advantage being simple",
                                                               "disadvantage: could dramatically increase volume of data unnecessarily transmitted to each Agent ",
                                                               "secretes encrypted for single host best inline",
                                                               "secretes encrypted for multiple hosts best in separate file"
                                                              ],
                                                  "description": "The name of the host" }
                                            }
            },

            "classes": {
                "My_other_class": {
                    "meta": { "comment": "i love this one", "description": "so much meta" }
                }
            }
        }
    ],
    "meta": {
        "count": 1,
        "page": 1,
        "MOST_RECENT_CMDB_CHANGE": 1617119492,
        "timestamp": 1617119889,
        "total": 1
    }
}
';

      "_d_data_idx" slist => getindices( "_d[data]" );

  files:
      "/tmp/$(with)"
        create => "true",
        with => "$(_d[data][$(_d_data_idx)]).json",
        template_data => mergedata("_d[data][$(_d_data_idx)]" ),
        template_method => "inline_mustache",
        edit_template_string => string_mustache( "{{$-top-}}", mergedata("_d[data][$(_d_data_idx)]" ) );
  reports: "Running CFEngine $(sys.cf_version)";
   "Record for $(with)"
     with => "$(_d[data][$(_d_data_idx)])",
     printfile => cat( "/tmp/$(_d[data][$(_d_data_idx)]).json" );
}
```

First Run

``` example
    info: Created file '/tmp/SHA=50d3ca3f9f0bcf02e2386ff86240bbc0d63e09764db052f97fec92eb84ecf449.json', mode 0600
    info: Updated rendering of '/tmp/SHA=50d3ca3f9f0bcf02e2386ff86240bbc0d63e09764db052f97fec92eb84ecf449.json' from mustache template 'inline'
    info: files promise '/tmp/SHA=50d3ca3f9f0bcf02e2386ff86240bbc0d63e09764db052f97fec92eb84ecf449.json' repaired
    info: Created file '/tmp/SHA=40d3ca3f9f0bcf02e2386ff86240bbc0d63e09764db052f97fec92eb84ecf449.json', mode 0600
    info: Updated rendering of '/tmp/SHA=40d3ca3f9f0bcf02e2386ff86240bbc0d63e09764db052f97fec92eb84ecf449.json' from mustache template 'inline'
    info: files promise '/tmp/SHA=40d3ca3f9f0bcf02e2386ff86240bbc0d63e09764db052f97fec92eb84ecf449.json' repaired
R: Running CFEngine 3.17.0
R: Record for SHA=50d3ca3f9f0bcf02e2386ff86240bbc0d63e09764db052f97fec92eb84ecf449
R: {"classes":{"My_class":{"meta":{"comment":"i love this one","description":"so much meta"}}},"id":"SHA=50d3ca3f9f0bcf02e2386ff86240bbc0d63e09764db052f97fec92eb84ecf449","vars":{"HubCMDB:My.hostname":{"meta":{"comment":"My hostname should be set to this","description":"The name of the host"},"value":"host1.cfengine.com"},"HubCMDB:My.pypy_password":{"meta":{"comments":["inline secret advantage being simple","disadvantage: could dramatically increase volume of data unnecessarily transmitted to each Agent ","secretes encrypted for single host best inline","secretes encrypted for multiple hosts best in separate file"],"description":"The name of the host"},"secret":"*($@*&%(&$#@))","secret_file":"secret1.cfsecret"},"Namespace:BundleName.VariableName":{"meta":{"comment":"def.json will not overwrite variables set by cmdb data, this applies to data files by augments key","derived_from":"host_specific.json (inserted by agent automatically)","description":"so much meta"},"value":"myvalue"}}}
R: Record for SHA=40d3ca3f9f0bcf02e2386ff86240bbc0d63e09764db052f97fec92eb84ecf449
R: {"classes":{"My_other_class":{"meta":{"comment":"i love this one","description":"so much meta"}}},"id":"SHA=40d3ca3f9f0bcf02e2386ff86240bbc0d63e09764db052f97fec92eb84ecf449","vars":{"HubCMDB:My.hostname":{"meta":{"comment":"My hostname should be set to this","description":"The name of the host"},"value":"host2.cfengine.com"},"HubCMDB:My.pypy_password":{"meta":{"comments":["inline secret advantage being simple","disadvantage: could dramatically increase volume of data unnecessarily transmitted to each Agent ","secretes encrypted for single host best inline","secretes encrypted for multiple hosts best in separate file"],"description":"The name of the host"},"secret":"*($@*&%(&$#@))","secret_file":"secret2.cfsecret"},"Namespace:BundleName.VariableName":{"meta":{"comment":"def.json will not overwrite variables set by cmdb data, this applies to data files by augments key","derived_from":"host_specific.json (inserted by agent automatically)","description":"so much meta"},"value":"myvalue"}}}
```

Second Run

``` example
R: Running CFEngine 3.17.0
R: Record for SHA=50d3ca3f9f0bcf02e2386ff86240bbc0d63e09764db052f97fec92eb84ecf449
R: {"classes":{"My_class":{"meta":{"comment":"i love this one","description":"so much meta"}}},"id":"SHA=50d3ca3f9f0bcf02e2386ff86240bbc0d63e09764db052f97fec92eb84ecf449","vars":{"HubCMDB:My.hostname":{"meta":{"comment":"My hostname should be set to this","description":"The name of the host"},"value":"host1.cfengine.com"},"HubCMDB:My.pypy_password":{"meta":{"comments":["inline secret advantage being simple","disadvantage: could dramatically increase volume of data unnecessarily transmitted to each Agent ","secretes encrypted for single host best inline","secretes encrypted for multiple hosts best in separate file"],"description":"The name of the host"},"secret":"*($@*&%(&$#@))","secret_file":"secret1.cfsecret"},"Namespace:BundleName.VariableName":{"meta":{"comment":"def.json will not overwrite variables set by cmdb data, this applies to data files by augments key","derived_from":"host_specific.json (inserted by agent automatically)","description":"so much meta"},"value":"myvalue"}}}
R: Record for SHA=40d3ca3f9f0bcf02e2386ff86240bbc0d63e09764db052f97fec92eb84ecf449
R: {"classes":{"My_other_class":{"meta":{"comment":"i love this one","description":"so much meta"}}},"id":"SHA=40d3ca3f9f0bcf02e2386ff86240bbc0d63e09764db052f97fec92eb84ecf449","vars":{"HubCMDB:My.hostname":{"meta":{"comment":"My hostname should be set to this","description":"The name of the host"},"value":"host2.cfengine.com"},"HubCMDB:My.pypy_password":{"meta":{"comments":["inline secret advantage being simple","disadvantage: could dramatically increase volume of data unnecessarily transmitted to each Agent ","secretes encrypted for single host best inline","secretes encrypted for multiple hosts best in separate file"],"description":"The name of the host"},"secret":"*($@*&%(&$#@))","secret_file":"secret2.cfsecret"},"Namespace:BundleName.VariableName":{"meta":{"comment":"def.json will not overwrite variables set by cmdb data, this applies to data files by augments key","derived_from":"host_specific.json (inserted by agent automatically)","description":"so much meta"},"value":"myvalue"}}}
```
