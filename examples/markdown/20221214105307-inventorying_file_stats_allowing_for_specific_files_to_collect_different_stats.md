We need to avoid collecting for the files that have specific attributes
requested when collecting for the files using the defaults. Then, for
the files that have specific overrides, we collect the attributes
requested.

:::: captioned-content
::: caption
Example Policy
:::

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both" extra-opts="--show-evaluated-vars=default:test\\\\." tangle="inventorying_file_stats_allowing_for_specific_files_to_collect_different_stats.cf"}
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

      "file_path" slist => { #"/tmp/file1.txt", # specific files could exist in this list or not, they are explicity accounted for
                             "/tmp/file2.txt"};

      # Define the default set of attributes to collect if the variable is not
      # already defined with the expectation that if it's already defined,
      # that's because it was pre-defined via Augments (def.json, or
      # host_specific.json)

      "attributes"
        slist => string_split("size,mtime,modeoct,permsoct,gid,uid", ",", 999), # Rather than actually splitting this, id just define it as a list and save the clock tick on the function execution.
        if => not( isvariable( "attributes" ) );

    # default:test.file_attr - data containerwith keys of file names and values of the filestat info to collect for each file
      "_attr_override_files"
        slist => getindices( file_attr ),
        if => isvariable( file_attr );

      "file_info[$(file_path)][$(attributes)]"
        string => filestat( $(file_path), $(attributes) ),
        if => and( fileexists("$(file_path)"),
                   not( isvariable( "file_attr[$(file_path)]" ) ) ); # If file attributes are overridden for a specific file, we will do it separately

      "file_info[$(_attr_override_files)][$(file_attr[$(_attr_override_files)])]"
        string => filestat( "$(_attr_override_files)", "$(file_attr[$(_attr_override_files)])" ),
        if => fileexists("$(_attr_override_files)");
}
```
::::

Then, in augments we can request specific attributes for specific files.

``` {.json tangle="/tmp/def.json"}
{
    "variables": {
        "default:test.file_attr": {
            "value": {
                "/tmp/file1.txt": [ "size", "mtime" ]
                },
            "comment": "Specific file filestat attribute collection settings"
        }
    }
}
```

And, we can see that `/tmp/file1.txt`{.verbatim} got mtime and size
collected while `file2.txt`{.verbatim} had all of them.

``` {.bash org-language="sh" results="output" exports="both"}
chmod 600 /tmp/def.json /tmp/inventory-filestats-w-specific-file-overrides.cf
cf-agent -Kf /tmp/inventory-filestats-w-specific-file-overrides.cf --show-evaluated-vars=default:test\\.
```

``` example
Variable name                            Variable value                                               Meta tags                                Comment
default:test._attr_override_files         {"/tmp/file1.txt"}                                          source=promise
default:test.attributes                   {"size","mtime","modeoct","permsoct","gid","uid"}           source=promise
default:test.file_attr                   {"/tmp/file1.txt":["size","mtime"]}                          source=augments_file                     Specific file filestat attribute collection settings
default:test.file_info[/tmp/file1.txt][mtime] 1671032449                                                   source=promise
default:test.file_info[/tmp/file1.txt][size] 6                                                            source=promise
default:test.file_info[/tmp/file2.txt][gid] 1000                                                         source=promise
default:test.file_info[/tmp/file2.txt][modeoct] 100600                                                       source=promise
default:test.file_info[/tmp/file2.txt][mtime] 1671032449                                                   source=promise
default:test.file_info[/tmp/file2.txt][permsoct]                                                              source=promise
default:test.file_info[/tmp/file2.txt][size] 6                                                            source=promise
default:test.file_info[/tmp/file2.txt][uid] 1000                                                         source=promise
default:test.file_path                    {"/tmp/file2.txt"}                                          source=promise
```
