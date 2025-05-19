I want to watch a directory and trigger some action something in the
sub-tree changes.

# Using `changes`{.verbatim} body {#68927029-323d-4a03-bf48-5e8e8dc9dcb5}

This example illustrates watching `/tmp/test-dir`{.verbatim} including
infinitely descending sub-directories for signs of change.

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both"}
bundle agent __main__
{
  files:

      # When using a changes body with a promise that uses regex to match
      # multiple files or depth_search it's important to exclude CFEngines state
      # tracking and backup files.

      "/tmp/test-dir/."
        depth_search => recurse( inf ),
        file_select => not_automatically_created_cfengine_files,
        changes => watch_no_report,
        classes => results( "bundle", "watch_dir" );

  reports:
      "Classes: $(with)" with => join( ", ", classesmatching( "watch_dir_.*" ) );

}
body file_select not_automatically_created_cfengine_files
# @bref Find files that do not match patterns of CFEngines automatically created files
{
        leaf_name => { ".*_cfchanges", ".*cfsaved", ".*cfnew", ".*cf-before-edit" };
        file_result => "!leaf_name";
}
body changes watch_no_report
# @brief Detect content changes in (small) files
# and report the diff to CFEngine Enterprise
{
        hash           => "sha256";
        # report_changes => "none"; # Causes change monitoriong to only notice files/directories coming and going, but not changes to them.
        report_changes => "all"; # Criteria for change warnings (all|stats|content|none)
        report_diffs   => "false";
        update_hashes  => "yes";
}
```

run 13 (added subtree/file.txt)

```{=org}
#+RESULTS:
```
``` example
  notice: New file '/tmp/test-dir/./subtree/file.txt' found
    info: Recorded directory listing for '/tmp/test-dir/./subtree'
R: Classes: watch_dir_repaired, watch_dir_kept, watch_dir_reached
```

run 12 (removed subtree)

```{=org}
#+RESULTS:
```
``` example
  notice: File '/tmp/test-dir/./subtree' no longer exists
    info: Removal of '/tmp/test-dir/./subtree' recorded
R: Classes: watch_dir_repaired, watch_dir_kept, watch_dir_reached
```

run 12 (removed subtree/file.txt)

```{=org}
#+RESULTS:
```
``` example
  notice: File '/tmp/test-dir/./subtree/file.txt' no longer exists
    info: Removal of '/tmp/test-dir/./subtree/file.txt' recorded
    info: Recorded directory listing for '/tmp/test-dir/./subtree'
R: Classes: watch_dir_repaired, watch_dir_kept, watch_dir_reached
```

run 11 (added subtree/file.txt)

```{=org}
#+RESULTS:
```
``` example
  notice: New file '/tmp/test-dir/./subtree/file.txt' found
    info: Recorded directory listing for '/tmp/test-dir/./subtree'
  notice: New file '/tmp/test-dir/./subtree' found
  notice: File '/tmp/test-dir/./sub-dir' no longer exists
    info: Removal of '/tmp/test-dir/./sub-dir' recorded
    info: Recorded directory listing for '/tmp/test-dir/.'
R: Classes: watch_dir_repaired, watch_dir_kept, watch_dir_reached
```

PROBLEM Run 10 (added back subdir/new.txt but it was not noticed!)

```{=org}
#+RESULTS:
```
``` example
R: Classes: watch_dir_kept, watch_dir_reached
```

PROBLEM Run 9 (removal of sub-tree, note there is no info message about
removal of `subdir/new.txt`{.verbatim})

```{=org}
#+RESULTS:
```
``` example
  notice: File '/tmp/test-dir/./sub-dir' no longer exists
    info: Removal of '/tmp/test-dir/./sub-dir' recorded
R: Classes: watch_dir_repaired, watch_dir_kept, watch_dir_reached
```

Run 8 (no change)

```{=org}
#+RESULTS:
```
``` example
R: Classes: watch_dir_kept, watch_dir_reached
```

Run 7 (New sub-directory with new file)

```{=org}
#+RESULTS:
```
``` example
  notice: New file '/tmp/test-dir/./sub-dir/new.txt' found
    info: Recorded directory listing for '/tmp/test-dir/./sub-dir'
  notice: New file '/tmp/test-dir/./sub-dir' found
    info: Recorded directory listing for '/tmp/test-dir/.'
R: Classes: watch_dir_repaired, watch_dir_kept, watch_dir_reached
```

Run 6 (no change)

```{=org}
#+RESULTS:
```
``` example
R: Classes: watch_dir_kept, watch_dir_reached
```

Run 5 (delete file)

```{=org}
#+RESULTS:
```
``` example
  notice: File '/tmp/test-dir/./new.txt' no longer exists
    info: Removal of '/tmp/test-dir/./new.txt' recorded
    info: Recorded directory listing for '/tmp/test-dir/.'
R: Classes: watch_dir_repaired, watch_dir_kept, watch_dir_reached
```

Run 4 (no change)

```{=org}
#+RESULTS:
```
``` example
R: Classes: watch_dir_kept, watch_dir_reached
```

Run 3 (added new file)

```{=org}
#+RESULTS:
```
``` example
  notice: New file '/tmp/test-dir/./new.txt' found
    info: Recorded directory listing for '/tmp/test-dir/.'
R: Classes: watch_dir_repaired, watch_dir_kept, watch_dir_reached
```

Run 2 (no change)

```{=org}
#+RESULTS:
```
``` example
R: Classes: watch_dir_kept, watch_dir_reached
```

Run 1 (initialization)

```{=org}
#+RESULTS:
```
``` example
  notice: New file '/tmp/test-dir/./script.sh' found
    info: Recorded directory listing for '/tmp/test-dir/.'
R: Classes: watch_dir_repaired, watch_dir_kept, watch_dir_reached
```

## Reference from ENT-6080 about state of monitoring a directory tree for changes

Monitoring a directory tree for changes

CFEngine tracking files (~cfchanges~, .cfsaved, .cf-before-edit,
.cf-new) are stored next to monitored filesThis ties into CFE-3275 where
we want to be able to configure (in case someone is relying on current
placement) where these files are saved.

If body file~select~ is not used to exclude ~cfchanges~ suffixed files
then there will be an explosion of filesEach file will get a ~cfchanges~
suffixed copy, even other ~cfchanges~ files, resulting in exponential
growth.

If body file~select~ is not used to exclude .csaved, .cf-before-edit,
.cf-new suffixed files then many files will be tracked for change
un-necessarily.

Reporting diffs for each file in a tree is not directly supportedbody
changes report~diffs~ =\> \"true\" will emit an error if combined with
depth search. In order to report diffs of changes to files in a tree,
the files must be enumerated (e.g. findfiles()) and individually
monitored for change.This ties back in to ENT-6079. If I have to have 2
promises to monitor a directory tree for changes and report diffs then I
can easily end up reporting multiple content change records.

Monitor directory tree for changes, handling cfengine tracking files and
recursion

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both"}
bundle agent main
{
  methods:
    # Activate your custom policies here
    "changes_testing";
}

bundle agent changes_testing
{

vars:
    "_leaf_name_exclude_patterns"
      slist => {  ".*_cfchanges", ".*cfsaved", ".*cfnew", ".*cf-before-edit" };

    "_dir" string => "/tmp/changes-testing";
    "_files" slist => findfiles( "/tmp/changes-testing/*" );

   files:

     "$(_files)"
       changes => report_content_diffs,
       handle => "changes_report_changes_none_report_diffs_true_action_bg_if_plain_not_exculded",
       #action => bg( "0", "5" ),
       if => not( regcmp( ".*(_cfchanges|cfsaved|cfnew|cf-before-edit)", "$(_files)"));

      "$(_dir)"
        changes => report_all_changes_no_diffs,
        handle => "changes_report_changes_all_report_diffs_false_exclude_cftracking",
        depth_search => recurse("inf"),
        file_select => leaf_name_exclude( @(_leaf_name_exclude_patterns) );

}

body file_select leaf_name_exclude( excluded_leaf_names )
{
  leaf_name => { @(excluded_leaf_names) };
  file_result => "!leaf_name";
}


body changes report_content_diffs
# @brief Detect all file changes using sha256 and report the diff to CFEngine Enterprise
{
      hash           => "sha256";
      report_changes => "content"; # Required for report_diffs => "true"; to function properly;
      report_diffs   => "true";    # requires report_changes => "content|all";
      update_hashes  => "yes";
}

body changes report_all_changes_no_diffs
{
      hash           => "sha256";
      report_changes => "all";
      report_diffs   => "false";
      update_hashes  => "yes";
}
```
