``` {.cfengine3 include-stdlib="t" log-level="info" exports="both"}
bundle agent __main__
{

  vars:
      "config_file" string => "/tmp/config.txt";


  files:
      "$(config_file)"
        content => "my_service_config",
        classes => results( "bundle", "my_service_config" );

  services:
      "my_service"
        service_policy => "start";

    my_service_config_reapaired::
      "my_service"
        service_policy => "restart";
}
```

# With service config validation

Not every service provides the ability to validate a configuration file,
but for those that do (like `apache configcheck`, `visudo -c`,
`sshd -t`) this is a nice option.

``` {.cfengine3 include-stdlib="t" log-level="info" exports="both"}
bundle agent __main__
{

  vars:
      "config_file" string => "/tmp/config.txt";

  classes:
    my_service_staged_config_repaired::
      "my_service_staged_config_validated"
        expression => returnszero( "/bin/validation/cmd", noshell );

  files:
      "$(config_file).staged"
        copy_from => local_dcp( "$(config_file)" );

      "$(config_file).staged"
        content => "My config values",
        classes => results( "bundle", "my_service_staged_config" );

    my_service_staged_config_repaired.my_service_staged_config_validated::
       "$(config_file)"
        copy_from => local_dcp( "$(config_file).staged" );
        classes => results( "bundle", "my_service_config" );

  services:
      "my_service"
        service_policy => "start";

    my_service_config_reapaired::
      "my_service"
        service_policy => "restart",
        classes => results( "bundle", "my_service_restart" );
}
```
