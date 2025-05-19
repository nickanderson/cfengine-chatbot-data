the filters for getusers are comma separated strings.

``` {.cfengine3 tangle="getusers_filtering_out_a_subset_of_user_ids.cf"}
bundle agent __main__
{
  vars:
    "users" slist => getusers( "", "");
    "users_id_gt_999" slist => getusers( "", join( ",", expandrange( "[0-999]", 1 ) ) );
}
```

``` example
[root@hub masterfiles]# cf-agent -KIf ./example.cf --show-evaluated-vars=default:main\\.
Variable name                            Variable value                                               Meta tags                                Comment
default:main.users                        {"root","bin","daemon","adm","lp","sync","shutdown","halt","mail","operator","games","ftp","nobody","systemd-network","dbus","polkitd","rpc","tss","rpcuser","nfsnobody","sshd","postfix","chrony","vagrant","cfapache","cfpostgres"} source=promise
default:main.users_id_gt_999              {"nfsnobody","vagrant"}                                     source=promise
```
