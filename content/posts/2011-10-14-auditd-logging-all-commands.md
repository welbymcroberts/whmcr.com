+++
title = "Auditd logging all commands"
#description = ""
date = "2011-10-14T19:58:39+01:00"
#updated = ""
#slug = ""
weight = 0
draft = false
#path = 
aliases = [
    "/?p=251",
    "/2011/10/14/auditd-logging-all-commands/"
    
]
in_search_index = true


tags = [
    "Linux",
    "Projects",
    "Software"
]
[extra]
+++


A common requirement for PCI-DSS is for all commands run by a user who has admin privileges to be logged. There are many ways to do this, most of the time people will opt for a change to the bash configuration or to rely on sudo. There are many ways arround this (such as providing the command that you wish to run as a paramater to SSH). As the linux kernel provides a full auditing system. Using a utility such as auditd, we are able to log all commands that are run. The configuration for this is actually quite simple. In `/etc/audit/audit.rules` we need to ensure that the following exists.  
```
-a exit,always -F arch=b64 -S execve
-a exit,always -F arch=b32 -S execve
```

This will capture any execve system call (on exit) and will log this to the auditd log. A log entry will look similar to below.  
```
type=SYSCALL msg=audit(1318930500.123:3020171): arch=c000003e syscall=59 success=yes exit=0 a0=7fff65179def a1=7fff65179ec0 a2=7fff6517d060 a3=7ff54ee36c00 items=3 ppid=9200 pid=9202 auid=0 uid=1000 gid=100 euid=1000 suid=1000 fsuid=1000 egid=100 sgid=100 fsgid=100 tty=(none) ses=4 comm="xscreensaver-ge" exe="/usr/bin/perl" key=(null)
type=EXECVE msg=audit(1318930500.123:3020171): argc=5 a0="/usr/bin/perl" a1="-w" a2="/usr/bin/xscreensaver-getimage-file" a3="--name" a4="/home/welby/Pictures"
type=EXECVE msg=audit(1318930500.123:3020171): argc=3 a0="/usr/bin/perl" a1="-w" a2="/usr/bin/xscreensaver-getimage-file"
type=CWD msg=audit(1318930500.123:3020171):  cwd="/home/welby/Downloads"
type=PATH msg=audit(1318930500.123:3020171): item=0 name="/usr/bin/xscreensaver-getimage-file" inode=208346 dev=fe:02 mode=0100755 ouid=0 ogid=0 rdev=00:00
type=PATH msg=audit(1318930500.123:3020171): item=1 name=(null) inode=200983 dev=fe:02 mode=0100755 ouid=0 ogid=0 rdev=00:00
type=PATH msg=audit(1318930500.123:3020171): item=2 name=(null) inode=46 dev=fe:02 mode=0100755 ouid=0 ogid=0 rdev=00:00
```  
This should keep most auditors happy 🙂