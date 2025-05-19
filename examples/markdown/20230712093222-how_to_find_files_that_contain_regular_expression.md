<https://northerntech.atlassian.net/browse/CFE-4225>

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both" tangle="how_to_find_files_that_contain_regular_expression.cf"}
 bundle agent __main__
 {
   methods:
     "init";
     "test";
 }
bundle agent init
{
    vars:
      "range1" slist => expandrange( "[0-3]", 1 );
      "range2" slist => expandrange( "[4-7]", 1 );

    files:
      "/tmp/CFE-4225/dir-$(range1)/."
        create => "true";

      "/tmp/CFE-4225/dir-$(range1)/file-$(range1).txt"
        create => "true",
        content => concat(
                           "I've got a lovely bunch of coconuts.$(const.n)",
                           "managed by CFEngine$(const.n)",
                           "There they are standing in a row." );

      "/tmp/CFE-4225/dir-$(range1)/file-$(range2).txt"
        create => "true",
        content => concat(
                           "I've got a lovely bunch of coconuts.$(const.n)",
                           "There they are standing in a row." );
}
bundle agent test
{
    vars:
      # Let's find things we need to consider from a directory down
      "found" slist => findfiles( "/tmp/CFE-4225/**" );

      # The above findfiles() result will contian directories. Since we are only interested in files we can iterate over the found files and build up an associatiave array of the found files that are plain files.

      "file[$(found)]"
        string => "$(found)",
        if => isplain( $(found) );

      "files" slist => getindices( file );

    reports:
      "$(files) contains 'managed by CFEngine'"
        if => regline( ".*managed by CFEngine.*", $(files) );
}
```

``` example
    info: Updated file '/tmp/CFE-4225/dir-0/file-0.txt' with content 'I've got a lovely bunch of coconuts.
managed by CFEngine
There they are standing in a row.'
    info: Updated file '/tmp/CFE-4225/dir-1/file-1.txt' with content 'I've got a lovely bunch of coconuts.
managed by CFEngine
There they are standing in a row.'
    info: Updated file '/tmp/CFE-4225/dir-2/file-2.txt' with content 'I've got a lovely bunch of coconuts.
managed by CFEngine
There they are standing in a row.'
    info: Updated file '/tmp/CFE-4225/dir-3/file-3.txt' with content 'I've got a lovely bunch of coconuts.
managed by CFEngine
There they are standing in a row.'
    info: Updated file '/tmp/CFE-4225/dir-0/file-4.txt' with content 'I've got a lovely bunch of coconuts.
There they are standing in a row.'
    info: Updated file '/tmp/CFE-4225/dir-0/file-5.txt' with content 'I've got a lovely bunch of coconuts.
There they are standing in a row.'
    info: Updated file '/tmp/CFE-4225/dir-0/file-6.txt' with content 'I've got a lovely bunch of coconuts.
There they are standing in a row.'
    info: Updated file '/tmp/CFE-4225/dir-0/file-7.txt' with content 'I've got a lovely bunch of coconuts.
There they are standing in a row.'
    info: Updated file '/tmp/CFE-4225/dir-1/file-4.txt' with content 'I've got a lovely bunch of coconuts.
There they are standing in a row.'
    info: Updated file '/tmp/CFE-4225/dir-1/file-5.txt' with content 'I've got a lovely bunch of coconuts.
There they are standing in a row.'
    info: Updated file '/tmp/CFE-4225/dir-1/file-6.txt' with content 'I've got a lovely bunch of coconuts.
There they are standing in a row.'
    info: Updated file '/tmp/CFE-4225/dir-1/file-7.txt' with content 'I've got a lovely bunch of coconuts.
There they are standing in a row.'
    info: Updated file '/tmp/CFE-4225/dir-2/file-4.txt' with content 'I've got a lovely bunch of coconuts.
There they are standing in a row.'
    info: Updated file '/tmp/CFE-4225/dir-2/file-5.txt' with content 'I've got a lovely bunch of coconuts.
There they are standing in a row.'
    info: Updated file '/tmp/CFE-4225/dir-2/file-6.txt' with content 'I've got a lovely bunch of coconuts.
There they are standing in a row.'
    info: Updated file '/tmp/CFE-4225/dir-2/file-7.txt' with content 'I've got a lovely bunch of coconuts.
There they are standing in a row.'
    info: Updated file '/tmp/CFE-4225/dir-3/file-4.txt' with content 'I've got a lovely bunch of coconuts.
There they are standing in a row.'
    info: Updated file '/tmp/CFE-4225/dir-3/file-5.txt' with content 'I've got a lovely bunch of coconuts.
There they are standing in a row.'
    info: Updated file '/tmp/CFE-4225/dir-3/file-6.txt' with content 'I've got a lovely bunch of coconuts.
There they are standing in a row.'
    info: Updated file '/tmp/CFE-4225/dir-3/file-7.txt' with content 'I've got a lovely bunch of coconuts.
There they are standing in a row.'
R: /tmp/CFE-4225/dir-0/file-0.txt contains 'managed by CFEngine'
R: /tmp/CFE-4225/dir-3/file-3.txt contains 'managed by CFEngine'
R: /tmp/CFE-4225/dir-2/file-2.txt contains 'managed by CFEngine'
R: /tmp/CFE-4225/dir-1/file-1.txt contains 'managed by CFEngine'
```
