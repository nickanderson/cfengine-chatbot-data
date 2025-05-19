``` {.cfengine3 tangle="default_directory_permissions.cf" log-level="info"}
bundle agent __main__
{
  files:
      "/tmp/a/promised/directory/."
        delete => default:tidy,
        handle => "dir_not_present",
        comment => "It's important that the directory not exist in order to illustrate the default";

      "/tmp/a/promised/directory/."
        create => "true",
        depends_on => { "dir_not_present" };

  reports:
    "CFEngine $(sys.cf_version)";
    "Directory has $(with)"
      with => concat( "permoct='", filestat( "/tmp/a/promised/directory/.", "permoct"  ), "', ",
                      "permstr='", filestat( "/tmp/a/promised/directory/.", "permstr"  ), "'" );
}
```
