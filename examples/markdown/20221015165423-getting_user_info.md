``` {.cfengine3 tangle="getting_user_info.cf" extra-opts="--show-evaluated-vars=main\\\\."}
bundle agent __main__
{
  vars:
      "users" slist => getusers( "root,daemon", concat( join( ",", expandrange( "[3-999]", 1 ) ) ) );

      "gid[$(users)]" int => getgid( $(users) );
      "uid[$(users)]" int => getuid( $(users) );

      "userinfo_$(with)"
        data => getuserinfo( $(users) ),
        with => canonify( "$(users)" );

}
```
