``` {.cfengine3 tangle="deleting_files_based_on_filename_file_type_and_modification_time.cf"}
bundle agent example_deleting_old_files
{
  files:
      "/tmp/path/to/search"
        depth_search => recurse_with_base( "inf" ),
        file_select => my_selection_criteria,
        delete => tidy;
}
body file_select my_selection_criteria
{
        leaf_name => { "~git\d\d.*" }; # Filenames that start with ~gitNN where N is an integer
        mtime => irange(
                         # Modification time between 6 months ago and agent start time aka now()
                         ago( 0, # years
                              6, # months
                              0, # days
                              0, # hours
                              0, # minutes
                              0),# seconds
                         now() );
        file_types => { "plain", "symlink" };

      # File names that match, that are plain files or symlinks and not modified in the last 6 months;
        file_result => "leaf_name.file_types.!mtime";
}

bundle agent init
{
  commands:
      "/bin/mkdir -p /tmp/path/to/search/very/deep";
      "/bin/touch -t 202012101830 /tmp/path/to/search/very/~git20";
      "/bin/touch -t 202012101830 /tmp/path/to/search/very/git20";
      "/bin/touch -t 202012101830 /tmp/path/to/search/very/~git99";
      "/bin/touch -t 202012101830 /tmp/path/to/search/very/deep/~git30";

}
bundle agent __main__
{
  methods:
      "init";
      "example_deleting_old_files";
}
```
