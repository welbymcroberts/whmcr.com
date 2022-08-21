+++
title = 'sip.conf'
date = '2005-05-25T21:40:30+01:00'
url = "/asterisk/sip.conf"
aliases = [
    "/?page_id=83",
    "/asterisk/23/",
    "/old-content/asterisk/sip_conf/",
]
+++

```
[general]
port = 5060				; Port to bind to
bindaddr = 0.0.0.0		; Address to bind to

;bindaddr = 192.168.0.5
externip = 81.41.218.85	 Address that we're going to put in SIP messages if we're behind a NAT
localnet = 192.168.0.0         ; Internal NETWORK address
localmask = 255.255.255.0      ; Internal netmask
context=inbound-from-sip		; Default for incoming calls
allow=ulaw
allow=alaw
allow=gsm
allow=all

musiconhold=default



; welby's cisco phone
[6001]
type=friend
context=inbound-from-local
secret=password
host=dynamic
canreinvite=no
callgroup=1
pickupgroup=1
mailbox=6001
canreinvite=no
username=6001




;person2 ata port 1

[6002]
type=friend
secret=password
auth=md5
nat=no
host=dynamic
reinvite=no
canreinvite=no
qualify=1000
dtmfmode=rfc2833
disallow=all
allow=all
context=inbound-from-local
mailbox=6002

;person3 ata port 2

[6003]
type=friend
secret=password
auth=md5
nat=no
host=dynamic
reinvite=no
canreinvite=no
qualify=1000
dtmfmode=rfc2833
disallow=all
allow=all
context=inbound-from-local
mailbox=6003
```