``` {.cfengine3 include-stdlib="t" log-level="info" exports="both"}
bundle agent _etc_login_defs_ENCRYPT_METHOD
{
  meta:
      "tags" slist =>  { "autorun" };

  vars:
      "_etc_login_defs"
        string => "/etc/login.defs",
        # Allow this to be set by host_specific Augments
        # https://docs.cfengine.com/docs/3.18/reference-language-concepts-augments.html#host_specific-json
        unless => strcmp( "cmdb", getvariablemetatags( "$(this.promiser)", "source" ) );

  reports:
      # Print out the value of _etc_login_defs along with it's tags.
      "_etc_login_defs == $(_etc_login_defs), tags: '$(with)'"
        with => join( ", ", getvariablemetatags( _etc_login_defs ) ),
        if => isvariable( _etc_login_defs );
}
```

**References:**

- [CFEngine Examples](id:38277465-771a-4db4-983a-8dfd434b1aff)
- [getvariablemetatags()](id:b58cf542-43fd-4401-92bf-6d64fe4d1d10)
- [isvariable()](id:02720c30-efe9-4bb8-b360-fbf79886a13d)
- [join()](id:b91239e5-37fb-4d53-8335-9a38a16800ca)
- [strcmp()](id:a136eeee-4d49-4d15-afb7-0e8c1104d488)
