Our build system deposits assets in a directory where they wait to be
scanned by clamAV. Once clamAV completes the scan, assets are stored and
made avaialble for futher use.

After downsizing several vms, we noticed that doc builds were taking
longer than expected to become live. Investigating this, we found that
our queue of assets waiting for scan was getting large. We had assets
older than 3 hours in the directory. We wrote this policy to inventory
the queued assets and the modification time of the oldest asset. The
policy also records the age of the oldest asset in minutes and measures
that value over time so that we can see that the age of the queue
normally looks like.

``` {.cfengine3 include-stdlib="nil" log-level="info" exports="both" extra-opts="--show-evaluated-vars=default:main\\\\."}
bundle agent main
{
  vars:
      "path" string => "/var/tmp/clamscan/queue";
      "builds_queued_for_av_scan"
        slist => lsdir( $(path), "^(?!(\.$|\.\.$)).*", false ),
        meta => { "inventory", "attribute_name=Builds queued for av scan" };

      "mtime[$(builds_queued_for_av_scan)]"
        string => filestat( "$(path)/$(builds_queued_for_av_scan)",
                            "mtime" );
      "ctime_of_oldest"
        string => nth( sort( getvalues( mtime ), int), 0 );

      "now" int => now();

      "age_of_oldest_in_m"
        string => format( "%d", eval( "($(now)-$(ctime_of_oldest))/60", math, infix ));

      "ictime_of_oldest"
        string => strftime( "gmtime", "%Y-%m-%d %T", $(ctime_of_oldest) ),
        meta => { "inventory", "attribute_name=Date of oldest build queued for av scan" };

  files:
      "$(sys.statedir)/age_of_oldest_job_queued_for_av_scan.txt"
        create => "true",
        edit_line => lines_present( $(age_of_oldest_in_m) ),
        edit_defaults => empty;
}
```

``` example
Variable name                            Variable value                                               Meta tags                               
default:main.age_of_oldest_in_s          2411011                                                      source=promise                          
default:main.builds_queued_for_av_scan    {"cfengine_support_case_-2018-11-02-1213","cfengine_support_case_-2018-11-02-1216","usr","run","cfengine_support_case_-2018-11-02-1218","dev","vmlinuz","home","cfengine_support_case_-2018-11-02-1215","lib64","sbin","srv","initrd.img","lost+found","cfengine_support_case_-2018-11-02-1230","mnt","cfengine_support_case_-2018-11-02-1221","sys","opt","TESTING","media","tmp","etc","cf_lock.log","cfengine_support_case_-2018-11-02-1212","northern.tech","lib","cfengine_support_case_-2018-11-02-1223","acltest","cfengine_support_case_-2018-11-02-1214","cfengine_support_case_-2018-11-02-1219","net","tar","boot","cdrom","cf_lastseen.log","cfengine_support_case_-2018-11-02-1238","root","initrd.img.old","var","vmlinuz.old","cfengine_support_case_-2018-11-02-1235","proc","cfengine_support_case_-2018-11-02-1224","snap","bin"} source=promise,inventory,attribute_name=Builds queued for av scan
default:main.ctime_of_oldest             1429704111                                                   source=promise                          
default:main.ictime_of_oldest            2015-04-22 12:01:51                                          source=promise,inventory,attribute_name=Date of oldest build queued for av scan
default:main.mtime[TESTING]              1541538248                                                   source=promise                          
default:main.mtime[acltest]              1510326564                                                   source=promise                          
default:main.mtime[bin]                  1574090198                                                   source=promise                          
default:main.mtime[boot]                 1574100612                                                   source=promise                          
default:main.mtime[cdrom]                1437710256                                                   source=promise                          
default:main.mtime[cf_lastseen.log]      1503331079                                                   source=promise                          
default:main.mtime[cf_lock.log]          1503331079                                                   source=promise                          
default:main.mtime[cfengine_support_case_-2018-11-02-1212] 1541178770                                                   source=promise                          
default:main.mtime[cfengine_support_case_-2018-11-02-1213] 1541178801                                                   source=promise                          
default:main.mtime[cfengine_support_case_-2018-11-02-1214] 1541178850                                                   source=promise                          
default:main.mtime[cfengine_support_case_-2018-11-02-1215] 1541178900                                                   source=promise                          
default:main.mtime[cfengine_support_case_-2018-11-02-1216] 1541178981                                                   source=promise                          
default:main.mtime[cfengine_support_case_-2018-11-02-1218] 1541179084                                                   source=promise                          
default:main.mtime[cfengine_support_case_-2018-11-02-1219] 1541179154                                                   source=promise                          
default:main.mtime[cfengine_support_case_-2018-11-02-1221] 1541179294                                                   source=promise                          
default:main.mtime[cfengine_support_case_-2018-11-02-1223] 1541179393                                                   source=promise                          
default:main.mtime[cfengine_support_case_-2018-11-02-1224] 1541179607                                                   source=promise                          
default:main.mtime[cfengine_support_case_-2018-11-02-1230] 1541179940                                                   source=promise                          
default:main.mtime[cfengine_support_case_-2018-11-02-1235] 1541180116                                                   source=promise                          
default:main.mtime[cfengine_support_case_-2018-11-02-1238] 1541180318                                                   source=promise                          
default:main.mtime[dev]                  1574100337                                                   source=promise                          
default:main.mtime[etc]                  1574103802                                                   source=promise                          
default:main.mtime[home]                 1558533258                                                   source=promise                          
default:main.mtime[initrd.img.old]       1574098457                                                   source=promise                          
default:main.mtime[initrd.img]           1574098457                                                   source=promise                          
default:main.mtime[lib64]                1555947338                                                   source=promise                          
default:main.mtime[lib]                  1555949569                                                   source=promise                          
default:main.mtime[lost+found]           1437710124                                                   source=promise                          
default:main.mtime[media]                1558533258                                                   source=promise                          
default:main.mtime[mnt]                  1543671784                                                   source=promise                          
default:main.mtime[net]                  1574100263                                                   source=promise                          
default:main.mtime[northern.tech]        1523460078                                                   source=promise                          
default:main.mtime[opt]                  1556557556                                                   source=promise                          
default:main.mtime[proc]                 1574100209                                                   source=promise                          
default:main.mtime[root]                 1572359493                                                   source=promise                          
default:main.mtime[run]                  1574170906                                                   source=promise                          
default:main.mtime[sbin]                 1571332789                                                   source=promise                          
default:main.mtime[snap]                 1571329168                                                   source=promise                          
default:main.mtime[srv]                  1429704111                                                   source=promise                          
default:main.mtime[sys]                  1574100223                                                   source=promise                          
default:main.mtime[tar]                  1541174380                                                   source=promise                          
default:main.mtime[tmp]                  1574364694                                                   source=promise                          
default:main.mtime[usr]                  1540403385                                                   source=promise                          
default:main.mtime[var]                  1553975032                                                   source=promise                          
default:main.mtime[vmlinuz.old]          1574098457                                                   source=promise                          
default:main.mtime[vmlinuz]              1574098457                                                   source=promise                          
default:main.now                         1574364776                                                   source=promise                          
default:main.path                        /                                                            source=promise                          
```
