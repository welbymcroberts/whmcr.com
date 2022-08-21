+++
title = 'manager.conf'
date = '2005-05-25T21:40:30+01:00'
url = "/asterisk/manager.conf"
aliases = [
    "/?page_id=71",
    "/asterisk/17/",
    "/old-content/asterisk/manager_conf/",
]
+++
```
[general]
	enabled = no
	port = 5038
	bindaddr = 0.0.0.0
[user]
secret = pass
permit = 192.168.101.0/255.255.255.0
read = system,call,log,verbose,command,agent,user
write = system,call,log,verbose,command,agent,user
```