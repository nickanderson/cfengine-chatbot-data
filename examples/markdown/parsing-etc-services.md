``` cfengine3
bundle agent main
{
  vars:
    "services" string => "tcp 127.0.0.1:25  1147/master
tcp 0.0.0.0:443  1039/nginx:
tcp 127.0.0.1:8001  1218/python
tcp 0.0.0.0:10050  939/zabbix_agentd
tcp 127.0.0.1:6379  891/redis-server
tcp 0.0.0.0:80  1039/nginx:
tcp 0.0.0.0:22  889/sshd
tcp 127.0.0.1:5432  929/postmaster
udp 127.0.0.1:323 645/chronyd";

    "p" int => parsestringarrayidx(array, $(services), "", "[:/\s]", inf, inf);
    "i" slist => getindices( array ); 
    reports:
       "$(with)" with => string_mustache( "{{%-top-}}", mergedata( array )); 
       "$(array[$(i)][0]) $(array[$(i)][2]) $(array[$(i)][5])";
}
```
