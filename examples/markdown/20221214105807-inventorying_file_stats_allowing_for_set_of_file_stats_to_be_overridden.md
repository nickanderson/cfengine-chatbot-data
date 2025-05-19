``` {.cfengine3 include-stdlib="t" log-level="info" exports="both" extra-opts="--show-evaluated-vars=default:test\\\\." tangle="inventorying_file_stats_allowing_for_set_of_file_stats_to_be_overridden.cf"}
bundle agent __main__
{
  methods:
      "init";
      "test";
}
bundle agent init
{
  files:
      "/tmp/file1.txt" content => "file 1";
      "/tmp/file2.txt" content => "file 2";
}
bundle agent test
{
  vars:

    any::

      "file_path" slist => { "/tmp/file1.txt",
                             "/tmp/file2.txt"};

      # Define the default set of attributes to collect if the variable is not
      # already defined with the expectation that if it's already defined,
      # that's because it was pre-defined via Augments (def.json, or
      # host_specific.json)

      "attributes"
        slist => string_split("size,mtime,modeoct,permsoct,gid,uid", ",", 999),
        if => not( isvariable( "attributes" ) );

      "file_info[$(file_path)][$(attributes)]"
        string => filestat( $(file_path), $(attributes) ),
        if => fileexists("$(file_path)");
}
```

``` example
Variable name                            Variable value                                               Meta tags                                Comment
default:test.attributes                   {"size","mtime","modeoct","permsoct","gid","uid"}           source=promise
default:test.file_info[/tmp/file1.txt][gid] 1000                                                         source=promise
default:test.file_info[/tmp/file1.txt][modeoct] 100600                                                       source=promise
default:test.file_info[/tmp/file1.txt][mtime] 1671032449                                                   source=promise
default:test.file_info[/tmp/file1.txt][permsoct]                                                              source=promise
default:test.file_info[/tmp/file1.txt][size] 6                                                            source=promise
default:test.file_info[/tmp/file1.txt][uid] 1000                                                         source=promise
default:test.file_info[/tmp/file2.txt][gid] 1000                                                         source=promise
default:test.file_info[/tmp/file2.txt][modeoct] 100600                                                       source=promise
default:test.file_info[/tmp/file2.txt][mtime] 1671032449                                                   source=promise
default:test.file_info[/tmp/file2.txt][permsoct]                                                              source=promise
default:test.file_info[/tmp/file2.txt][size] 6                                                            source=promise
default:test.file_info[/tmp/file2.txt][uid] 1000                                                         source=promise
default:test.file_path                    {"/tmp/file1.txt","/tmp/file2.txt"}                         source=promise
```

And, say we have an augments file ...

``` {.json tangle="no"}
{
    "variables": {
        "default:test.attributes": {
            "value": [ "size", "modeoct" ],
            "comment": "The file attribtues that we wish to collect instead of the defaults"
        }
    }
}
```

We can see that the list of attributes differs.

``` {.bash org-language="sh" results="output" exports="both"}
chmod 600 /tmp/def.json /tmp/inventory-filestats.cf
cf-agent -Kf /tmp/inventory-filestats.cf --show-evaluated-vars=default:test\\.
```

``` example
Variable name                            Variable value                                               Meta tags                                Comment
default:test.attributes                   {"size","modeoct"}                                          source=augments_file                     The file attribtues that we wish to collect instead of the defaults
default:test.file_info[/tmp/file1.txt][modeoct] 100600                                                       source=promise
default:test.file_info[/tmp/file1.txt][size] 6                                                            source=promise
default:test.file_info[/tmp/file2.txt][modeoct] 100600                                                       source=promise
default:test.file_info[/tmp/file2.txt][size] 6                                                            source=promise
default:test.file_path                    {"/tmp/file1.txt","/tmp/file2.txt"}                         source=promise
```

It\'s not clear to me exactly how useful this would be in practice. Did
you plan to also have some sort of structure to try and target the list
of attributes for *specific files*? That seems like it would be more
useful than a global override unless of course you have multiple
environments where your global settings are different, then it could
make a lot of sense.

[Example illustrating inventorying file stats allowing for specific
files to collect different
stats](id:41252b2d-61f6-4418-bf53-b19b3481327e)
