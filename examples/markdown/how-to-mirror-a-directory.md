You can use the `purge`{.verbatim} attribute in body file control.

Here I create a structure:

``` bash
rm -rf /tmp/source
rm -rf /tmp/target
mkdir -p /tmp/source
mkdir -p /tmp/target
touch /tmp/source/present_in_source
touch /tmp/target/not_present_in_source
```

This policy will sync target with source making sure all files in source
have a copy in target and any files in target that are not in source are
purged.

``` {.cfengine3 exports="both"}
body agent control { inform => "true"; }

bundle agent main
{
  files:

      "/tmp/target/."
        copy_from => copyfrom_sync("/tmp/source"), # UGH Terrible body name in the stdlib
        file_select => all,
        depth_search => recurse_with_base(inf);
}
```

``` example
    info: Copying from 'localhost:/tmp/source/present_in_source'
    info: Purging '/tmp/target/./not_present_in_source' in copy dest directory
```

## How to purge when copying from a list

Yes, the `purge`{.verbatim} attribute in `copy_from`{.verbatim} bodies
does require a directory promise. It\'s useful when you don\'t know what
you are requesting other than a synchronization of directories.

However, if you are copying files based on a list, then you should have
complete knowledge of the files that should be present and from that you
can derive the files that should not be present.

Setup a tree: One file `present_in_source`{.verbatim} is in the source
directory, and one file `not_present_in_source`{.verbatim} is in the
target directory.

``` bash
rm -rf /tmp/source
rm -rf /tmp/target
mkdir -p /tmp/source
mkdir -p /tmp/target
touch /tmp/source/present_in_source
touch /tmp/target/not_present_in_source
```

This policy has a list of files that it promises to copy from the source
( `present_in_source`{.verbatim}, and
`some_other_file_not_present`{.verbatim} which doesnt actually exist in
the source ). It attempts to copy the list of files and then when done,
it gets a list of the files in that directory, finds the difference of
the files that we wanted to copy and what is in the target directory and
purges files not found in the list from the target directory.

``` {.cfengine3 exports="both"}
body agent control { inform => "true"; }
bundle agent main
{
  vars:
      "files" slist => { "present_in_source", "some_other_file_not_present" };

  files:
      "/tmp/target/$(files)"
        handle => "ensure_files_copy_from_source",
        copy_from => local_dcp( "/tmp/source/$(files)" );

  vars:
      "files_in_target_after_copy_from"
        depends_on => { "ensure_files_copy_from_source" },
        slist => lsdir( "/tmp/target/", ".*", "false" );

      "diff" slist => difference( files_in_target_after_copy_from, files);

  files:
      "/tmp/target/$(diff)"
        delete => tidy;
}
```

``` example
    info: Copying from 'localhost:/tmp/source/present_in_source'
    info: Can't stat file '/tmp/source/some_other_file_not_present' on 'localhost' in files.copy_from promise
    info: Deleted file '/tmp/target/not_present_in_source'
```
