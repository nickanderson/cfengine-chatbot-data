``` {.cfengine3 tangle="configuring_postfix_for_gmail_smtp.cf"}
bundle agent postfix_smtp
# Bundles that are added to bundlesequecne via augmetns cant take params. so this indierct wrapper
{

  methods:
      "" usebundle => postfix_smtp_gmail( "" );

}

bundle agent postfix_smtp_gmail(fqvarname)
{
      # Rather than take fqvarname as a param, we could have a var whos value is the fqvarname, settable via augments.

      # myhostname = hostname.example.com
      # relayhost = [smtp.gmail.com]:587
      # smtp_use_tls = yes
      # smtp_sasl_auth_enable = yes
      # smtp_sasl_password_maps = hash:/etc/postfix/sasl_password
      # smtp_tls_CAfile = /etc/ssl/certs/ca-bundle.crt
      # smtp_sasl_security_options = noanonymous
      # smtp_sasl_tls_security_options = noanonymous

  vars:
      "postfix_main_cf[relayhost]" string => "[smtp.gmail.com]:587";
      "postfix_main_cf[smtp_use_tls]" string => "yes";
      "postfix_main_cf[smtp_sasl_auth_enable]" string => "yes";
      "postfix_main_cf[smtp_sasl_password_maps]" string => "hash:/etc/postfix/sasl_password";
      "postfix_main_cf[smtp_tls_CAfile]" string => "/etc/ssl/certs/ca-bundle.crt";
      "postfix_main_cf[smtp_sasl_security_options]" string => "noanonymous";
      "postfix_main_cf[smtp_sasl_tls_security_options]" string => "noanonymous";

      "config"
        data => mergedata( postfix_main_cf, $(fqvarname) ),
        if => isvariable( $(fqvarname) );

      "config"
        data => mergedata( postfix_main_cf ),
        if => not( isvariable( $(fqvarname) ) );


  packages:
      "postfix" policy => "present";
      "cyrus-sasl-plain" policy => "present";
      "mailx" policy => "present";

  services:
      "postfix"
        service_policy => "start";

      "postfix"
        service_policy => "restart",
        handle => "postfix_restart_config_change",
        if => or( canonify("/etc/postfix/sasl_password.db_repaired"),
                  canonify("/etc/postfix/main.cf_repaired") );

  files:
      "/etc/postfix/main.cf"
        edit_line => default:set_line_based( "$(this.namespace):$(this.bundle).config",
                                             "=",
                                             "\s*=\s*",
                                             ".*",
                                             "\s*#\s*"),
        if => isdir( "/etc/postfix" ),
        classes => results( "bundle", "/etc/postfix/main.cf" );

      "/etc/postfix/sasl_password"
        perms => mog( 640, "root", "postfix" ),
        if => fileexists( "/etc/postfix/sasl_password" );

  commands:
      "/sbin/postmap"
        arglist => { "/etc/postfix/sasl_password" },
        handle => "postmap_db_absent",
        contain => in_shell,
        if => not( fileexists( "/etc/postfix/sasl_password.db" ) ),
        classes => results( "bundle", "/etc/postfix/sasl_password.db" );

      "/sbin/postmap"
        arglist => { "/etc/postfix/sasl_password" },
        handle => "postmap_source_newer_than_db",
        contain => in_shell,
        if => isnewerthan( "/etc/postfix/sasl_password",
                           "/etc/postfix/sasl_password.db" ),
        classes => results( "bundle", "/etc/postfix/sasl_password.db" );

  reports:
      "Missing '/etc/postfix/sasl_password'. Populate with '[smtp.gmail.com]:587 username:password'"
        if => not( fileexists( "/etc/postfix/sasl_password" ) );
}
```
