> \"Please describe how you would implement the following use case in
> your solution\'s domain-specific language, YAML etc. Target platforms:
> a generic UNIX-like operating system (POSIX).
>
> This compliance check passes if only root itself has write access to
> the files and directories (including symbolic links) in its \$PATH.
> How do you deal with LDAP/non-local user accounts?\"

We could get roots path and then promise that each path recursively has
files that are owned by root and that those files are only writeable by
root, not writeable by group and not writeable by others. In this
example warning vs fixed is implemented for easy switching by the value
of a variable. CFEngine operates on local files, and can promise the
permissions regardless if the user is from a directory or a local user.
We locate all symlinks that exist in any of the directories that make up
roots PATH, resolve those link targets and ensure they are root owned
and root writeable as well.

``` {.cfengine3 tangle="/tmp/test.cf" include-stdlib="no"}
bundle agent main
{
  vars:
      "mode" string => "warn";

    # Policy runs as root, so we execute env and extract PATH
      "PATH"
        string => execresult( "/usr/bin/env | awk -F '=' '/^PATH/ {print $2}'", useshell);

    # Here we split the PATH so that we can operate on each directory
      "PATHs" slist => string_split( "$(PATH)", ":", inf );


   # We need to locate all of the symlinks that reside in PATH and resolve the
   # link target for each.
    "symlinks[$(PATHs)]" string => execresult( "/usr/bin/find $(PATHs) -type l", noshell);
      "m" data => '[]';
      "m" data => mergedata( '[]', m, string_split( "$(symlinks[$(PATHs)])", "\n", inf) );
      "i" slist => getvalues( m );
    "target[$(i)]" string => filestat( $(i), linktarget );

  files:
    # Recursively search each directory in PATH aserting that all files (
    # including directories ) are owned by root, with user write access, without
    # group write access, without other write access. Warn or fix based on
    # $(mode) variable.

      # Here we ensure files in directories that makes up $PATH are owned, and
      # only writeable by root.

      "$(PATHs)/." -> { "CFG-072" }
        perms => mo( "u+w,g-w,o-w", root ),
        comment => "If non root users can modify files in roots PATH, then they
                    can subvert the system",
        depth_search => recurse_with_base( inf ),
        file_select => all,
        action => warn_or_fix( $(mode) );

      # Here we ensure that targets of symlinks found in $PATH are owned and
      # only writable by root

      "$(target[$(i)])" -> { "CFG-072" }
        comment => "If non root users can modify files that are targets of
                    symlinks in roots PATH, then they can subvert the system.",
        perms => mo( "u+w,g-w,o-w", root ),
        action => warn_or_fix( $(mode) );
}
body action warn_or_fix( mode )
{
        action_policy => "$(mode)";
}
# From stdlib ...
body file_select all
# @brief Select all file system entries
{
      leaf_name => { ".*" };
      file_result => "leaf_name";
}
body perms mo(mode,user)
# @brief Set the file's mode and owners
# @param mode The new mode
# @param user The username of the new owner
{
      owners => { "$(user)" };
      mode   => "$(mode)";
}
body depth_search recurse_with_base(d)
# @brief Search files and directories recursively up to the specified
# depth, starting from the base directory and including directories on
# other devices.
#
# @param d The maximum search depth
{
      depth => "$(d)";
      xdev  => "true";
      include_basedir => "true";
}


```

``` example
[root@hub vagrant]# cf-agent -KIf ./test.cf 
 warning: '/sbin/./dmeventd' has permission 0555 - [should be 0755]
 warning: '/sbin/./fsadm' has permission 0555 - [should be 0755]
 warning: '/sbin/./lvmdump' has permission 0555 - [should be 0755]
 warning: '/sbin/./blkdeactivate' has permission 0555 - [should be 0755]
 warning: '/sbin/./lvmconf' has permission 0555 - [should be 0755]
 warning: '/sbin/./lvm' has permission 0555 - [should be 0755]
 warning: '/sbin/./lvmetad' has permission 0555 - [should be 0755]
 warning: '/sbin/./vgimportclone' has permission 0555 - [should be 0755]
 warning: '/sbin/.' has permission 0555 - [should be 0755]
 warning: '/bin/.' has permission 0555 - [should be 0755]
 warning: '/usr/sbin/.' has permission 0555 - [should be 0755]
 warning: '/usr/bin/./sudo' has permission 4111 - [should be 4311]
 warning: '/usr/bin/./wall' has permission 2555 - [should be 2755]
 warning: '/usr/bin/./sudoreplay' has permission 0111 - [should be 0311]
 warning: '/usr/bin/.' has permission 0555 - [should be 0755]
```

### extras from development

``` example
R: /home/nickanderson/bin
R: /usr/local/sbin
R: /usr/local/bin
R: /usr/sbin
R: /usr/bin
R: /sbin
R: /bin
R: /usr/games
R: /usr/local/games
R: /snap/bin
R: /var/cfengine/bin
R: /usr/lib/jvm/java-8-oracle/bin
R: /usr/lib/jvm/java-8-oracle/db/bin
R: /usr/lib/jvm/java-8-oracle/jre/bin
```

``` example
/tmp/test.cf:15:0: error: Undefined body all with type file_select
   error: Policy failed validation with command '"/var/cfengine/bin/cf-promises" -c "/tmp/test.cf"'
   error: Failsafe condition triggered. Interactive session detected, skipping failsafe.cf execution.
   error: Error reading CFEngine policy. Exiting...
root@nickanderson-ThinkPad-W550s:~# chmod 600 /tmp/test.cf 
root@nickanderson-ThinkPad-W550s:~# cf-agent -KIf /tmp/test.cf 
    info: Owner of '/usr/local/bin/./ngrok' was 1000, setting to 0
    info: Group of '/usr/local/bin/./ngrok' was 1000, setting to 0
    info: Owner of '/usr/local/bin/./git-credential-netrc' was 1000, setting to 0
    info: Group of '/usr/local/bin/./git-credential-netrc' was 1000, setting to 0
    info: Object '/usr/local/bin/./git-credential-netrc' had permission 0775, changed it to 0755
 warning: NEW SETUID root PROGRAM '/usr/sbin/./pppd'
    info: Group of '/usr/sbin/./pppd' was 30, setting to 0
 warning: NEW SETGID root PROGRAM '/usr/sbin/./postdrop'
    info: Group of '/usr/sbin/./postdrop' was 143, setting to 0
    info: Object '/usr/sbin/./postdrop' had permission 2555, changed it to 2755
 warning: NEW SETGID root PROGRAM '/usr/sbin/./postqueue'
    info: Group of '/usr/sbin/./postqueue' was 143, setting to 0
    info: Object '/usr/sbin/./postqueue' had permission 2555, changed it to 2755
 warning: NEW SETUID root PROGRAM '/usr/bin/./pkexec'
 warning: NEW SETUID root PROGRAM '/usr/bin/./sudo'
 warning: NEW SETGID root PROGRAM '/usr/bin/./mlocate'
    info: Group of '/usr/bin/./mlocate' was 119, setting to 0
 warning: NEW SETGID root PROGRAM '/usr/bin/./bsd-write'
    info: Group of '/usr/bin/./bsd-write' was 5, setting to 0
 warning: NEW SETGID root PROGRAM '/usr/bin/./mail-unlock'
    info: Group of '/usr/bin/./mail-unlock' was 8, setting to 0
 warning: NEW SETUID root PROGRAM '/usr/bin/./gpasswd'
    info: Object '/usr/bin/./AfterShot3X64' had permission 0775, changed it to 0755
    info: Object '/usr/bin/./osqueryd' had permission 0775, changed it to 0755
 warning: NEW SETGID root PROGRAM '/usr/bin/./expiry'
    info: Group of '/usr/bin/./expiry' was 42, setting to 0
 warning: NEW SETGID root PROGRAM '/usr/bin/./jwhois'
    info: Group of '/usr/bin/./jwhois' was 137, setting to 0
 warning: NEW SETGID root PROGRAM '/usr/bin/./dotlockfile'
    info: Group of '/usr/bin/./dotlockfile' was 8, setting to 0
 warning: NEW SETGID root PROGRAM '/usr/bin/./chage'
    info: Group of '/usr/bin/./chage' was 42, setting to 0
 warning: NEW SETUID root PROGRAM '/usr/bin/./traceroute6.iputils'
    info: Object '/usr/bin/./atom' had permission 0775, changed it to 0755
 warning: NEW SETUID root PROGRAM '/usr/bin/./arping'
 warning: NEW SETGID root PROGRAM '/usr/bin/./ssh-agent'
    info: Group of '/usr/bin/./ssh-agent' was 120, setting to 0
 warning: NEW SETUID root PROGRAM '/usr/bin/./chsh'
    info: Object '/usr/bin/./osqueryctl' had permission 0775, changed it to 0755
 warning: NEW SETUID root PROGRAM '/usr/bin/./chfn'
 warning: NEW SETUID root PROGRAM '/usr/bin/./newgrp'
    info: Object '/usr/bin/./osqueryi' had permission 0775, changed it to 0755
 warning: NEW SETGID root PROGRAM '/usr/bin/./wall'
    info: Group of '/usr/bin/./wall' was 5, setting to 0
 warning: NEW SETGID root PROGRAM '/usr/bin/./crontab'
    info: Group of '/usr/bin/./crontab' was 109, setting to 0
 warning: NEW SETUID root PROGRAM '/usr/bin/./passwd'
 warning: NEW SETGID root PROGRAM '/sbin/./pam_extrausers_chkpwd'
    info: Group of '/sbin/./pam_extrausers_chkpwd' was 42, setting to 0
 warning: NEW SETGID root PROGRAM '/sbin/./unix_chkpwd'
    info: Group of '/sbin/./unix_chkpwd' was 42, setting to 0
 warning: NEW SETUID root PROGRAM '/sbin/./mount.nfs'
 warning: NEW SETUID root PROGRAM '/bin/./umount'
 warning: NEW SETUID root PROGRAM '/bin/./ping'
 warning: NEW SETUID root PROGRAM '/bin/./ntfs-3g'
 warning: NEW SETUID root PROGRAM '/bin/./mount'
 warning: NEW SETUID root PROGRAM '/bin/./su'
 warning: NEW SETUID root PROGRAM '/bin/./fusermount'
```
