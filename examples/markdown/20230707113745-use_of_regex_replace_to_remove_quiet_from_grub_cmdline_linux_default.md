Use `regex_replace()` to remove `quiet`{.verbatim} from
`GRUB_CMDLINE_LINUX_DEFAULT`{.verbatim}.

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both"}
bundle agent __main__
{
    methods:
     "init";
     "test";
}
bundle agent init
{
    files:
      "/tmp/test.txt"
        content => 'GRUB_CMDLINE_LINUX_DEFAULT="hobgoblin quiet splash bobgoblin"';
}
bundle agent test
{
   files:
      "/tmp/test.txt"
        content => regex_replace( readfile( "/tmp/test.txt", inf ),
                                  '^(GRUB_CMDLINE_LINUX_DEFAULT=".*)quiet(.*")', "\1\2", "");

   reports:
      "/tmp/test:" printfile => cat( "/tmp/test.txt" );

}
```

``` example
    info: Updated file '/tmp/test.txt' with content 'GRUB_CMDLINE_LINUX_DEFAULT="hobgoblin quiet splash bobgoblin"'
    info: Updated file '/tmp/test.txt' with content 'GRUB_CMDLINE_LINUX_DEFAULT="hobgoblin  splash bobgoblin"'
R: /tmp/test:
R: GRUB_CMDLINE_LINUX_DEFAULT="hobgoblin  splash bobgoblin"
```
