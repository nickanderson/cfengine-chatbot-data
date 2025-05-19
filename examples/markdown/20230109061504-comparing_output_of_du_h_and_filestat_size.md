``` {.cfengine3 exports="both" tangle="comparing_output_of_du_h_and_filestat_size.cf"}
bundle agent main
{
  files:
    "/tmp/file1.txtt"
      create => "true";

    "/tmp/file2.txtt"
      create => "true",
      edit_line => insert_lines("hello");

  vars:
    "files" slist => findfiles("/tmp/*.txtt");
    "cfiles[$(files)]" string => canonifyuniquely($(files));
    "sizes[$(cfiles[$(files)])]" string => filestat($(files), "size");
    "size_values" slist => getvalues(sizes);
    "total_size" real => sum(size_values);

    "duh[$(cfiles[$(files)])]" string => execresult("/usr/bin/du -h $(files)", noshell);
    "dub[$(cfiles[$(files)])]" string => execresult("/usr/bin/du -b $(files)", noshell);
    "duk[$(cfiles[$(files)])]" string => execresult("/usr/bin/du -k $(files)", noshell);

 reports:
  "CFEngine $(sys.cf_version)";
  "Total: $(total_size)";
  "du -h = $(duh[$(cfiles[$(files)])])";
  "du -b = $(dub[$(cfiles[$(files)])])";
  "du -k = $(duk[$(cfiles[$(files)])])";
}
```

``` example
: R: CFEngine 3.9.1
: R: Total: 12.000000
: R: du -h = 4.0K   /tmp/file1.txtt
: R: du -h = 4.0K   /tmp/file2.txtt
: R: du -b = 6  /tmp/file1.txtt
: R: du -b = 6  /tmp/file2.txtt
: R: du -k = 4  /tmp/file1.txtt
: R: du -k = 4  /tmp/file2.txtt
```
