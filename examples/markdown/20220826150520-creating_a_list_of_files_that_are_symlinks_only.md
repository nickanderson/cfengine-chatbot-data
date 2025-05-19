:::: captioned-content
::: caption
Example Policy
:::

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both" tangle="creating_a_list_of_files_that_are_symlinks_only.cf"}
bundle agent __main__ {
  methods:

      "First, we need symlinks to exist"
        usebundle => "init";

      "Then, we can find them using"
        usebundle => "find";

      "Finally, we can cleanup"
        usebundle => "cleanup";

}
bundle agent init
{
  vars:
      "symlinks"
        slist => { "one", "two", "three" };

  files:
      "/tmp/$(symlinks).ln"
        link_from => ln_s( "$(this.promise_filename)" );

}
bundle agent find
{
  vars:
    # Find files
      "found"
        slist => lsdir( "/tmp", ".*", "true" );

      # There may be other ways to achive this as well but using a classic
      # associative array and then pulling into a single list with getindices() or getvalues()
      # seemd natural to me.

      "link[$(found)]"
        string => "$(found)",
        if => islink( $(found) ); # You could also use filestat and strcmp for this

    # Get a consolidated list of symlinks from the associateve array.
      "links" slist => getindices( link );

  reports:

      "$(links) is a $(with)"
        with => concat( filestat( $(links), type ),
                        " to ",
                        filestat( $(links), linktarget ) );
}
bundle agent cleanup
{
  files:
      "$(find.links)" delete => tidy;
}
```
::::

``` example
    info: Linked files '/tmp/one.ln' -> '/home/nickanderson/org/roam/daily/work/cfengine3-DdnPlg'
    info: Linked files '/tmp/two.ln' -> '/home/nickanderson/org/roam/daily/work/cfengine3-DdnPlg'
    info: Linked files '/tmp/three.ln' -> '/home/nickanderson/org/roam/daily/work/cfengine3-DdnPlg'
R: /tmp/three.ln is a symlink to /home/nickanderson/org/roam/daily/work/cfengine3-DdnPlg
R: /tmp/two.ln is a symlink to /home/nickanderson/org/roam/daily/work/cfengine3-DdnPlg
R: /tmp/one.ln is a symlink to /home/nickanderson/org/roam/daily/work/cfengine3-DdnPlg
    info: Deleted file '/tmp/three.ln'
    info: Deleted file '/tmp/two.ln'
    info: Deleted file '/tmp/one.ln'
```
