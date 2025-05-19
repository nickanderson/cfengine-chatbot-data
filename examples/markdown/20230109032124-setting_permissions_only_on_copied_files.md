``` {.cfengine3 tangle="setting_permissions_only_on_copied_files.cf"}
bundle agent main
{

  methods:
    policy_server::
      "Keep profile.d manifest fresh"
        usebundle => srv_profile_d_manifest;

    any::
       "Populate profile.d"
         usebundle => profile_d;
}

bundle agent srv_profile_d_manifest
{
  vars:

    policy_server::

      "files" slist => lsdir("/srv/cfengine3/profile.d", ".*", "false");

  files:

    policy_server::

      "/srv/cfengine3/profile.d.manifest"
        create => "true",
        edit_defaults => empty,
        edit_line => insert_lines( @(files) ),
        comment => "We generate a manifest of the files found in the profile.d
                    directory so that remote hosts can know specifically what
                    files are available.";
}

bundle agent profile_d
{
  vars:

      "cached_manifest"
        string => "/tmp/profile.d.manifest",
        comment => "Where we should store our local copy of the profile.d
                    manifest.";

  files:

      "$(cached_manifest)"
        create => "true",
        copy_from => remote_dcp("/srv/cfengine3/profile.d.manifest", $(sys.policy_hub) ),
        comment => "We need to try and cache the manifest locally so that we can
                    read the list of files it contains.";

  vars:

      "manifest_files"
        slist => readstringlist( $(cached_manifest),
                                 "",
                                 "\n",
                                 100,
                                 100K),
        ifvarclass => fileexists( $(cached_manifest) ),
        comment => "Get the list of files from our cached manifest if it exists.";


  files:
      "/etc/profile.d"
        copy_from   => copy("pub_cfinput/profile.d/$(manifest_files)"),
        perms   => system_owned(0444);
}
```
