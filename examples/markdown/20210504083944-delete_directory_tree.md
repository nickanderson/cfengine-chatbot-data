This example illustrates how to delete a directory tree recursively.

:::: captioned-content
::: caption
:::

``` {.cfengine3 include-stdlib="nil" log-level="info" exports="both"}
bundle agent __main__
{
  vars:
    "test_files" slist => { "1", "2", "subdir/1", "subdir/3/4/5/file" };

  files:
      "/tmp/subdir/$(test_files)" create => "true";

      "/tmp/subdir/."
        depth_search => recurse( inf ),
        file_select => all,
        delete => my_tidy;

      "/tmp/subdir/."
        delete => my_tidy;
}
body file_select all
# @brief Select all file system entries
{
      leaf_name => { ".*" };
      file_result => "leaf_name";
}
body delete my_tidy
# @brief Delete the file and remove empty directories
# and links to directories
{
        dirlinks => "delete";
        rmdirs   => "true";
}
body depth_search recurse(d)
# @brief Search files and direcories recursively, up to the specified depth
# Directories on different devices are excluded.
#
# @param d The maximum search depth
{
      depth => "$(d)";
      xdev  => "true";
}
```
::::

``` example
info: Created directory for '/tmp/subdir/1'
info: Created file '/tmp/subdir/1', mode 0600
info: files promise '/tmp/subdir/1' repaired
info: Created file '/tmp/subdir/2', mode 0600
info: files promise '/tmp/subdir/2' repaired
info: Created directory for '/tmp/subdir/subdir/1'
info: Created file '/tmp/subdir/subdir/1', mode 0600
info: files promise '/tmp/subdir/subdir/1' repaired
info: Created directory for '/tmp/subdir/subdir/3/4/5/file'
info: Created file '/tmp/subdir/subdir/3/4/5/file', mode 0600
info: files promise '/tmp/subdir/subdir/3/4/5/file' repaired
info: Deleted file '/tmp/subdir/./2'
info: Deleted file '/tmp/subdir/./subdir/1'
info: Deleted file '/tmp/subdir/./subdir/3/4/5/file'
info: Deleted directory '/tmp/subdir/./subdir/3/4/5'
info: Deleted directory '/tmp/subdir/./subdir/3/4'
info: Deleted directory '/tmp/subdir/./subdir/3'
info: Deleted directory '/tmp/subdir/./subdir'
info: Deleted file '/tmp/subdir/./1'
info: files promise '/tmp/subdir/.' repaired
info: Deleted directory '/tmp/subdir'
info: files promise '/tmp/subdir' repaired
```
