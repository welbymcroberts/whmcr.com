+++
title = 'iax.conf'
date = '2005-05-25T21:40:30+01:00'
url = "/asterisk/iax.conf"
aliases = [
    "/?page_id=65",
    "/asterisk/14/",
    "/old-content/asterisk/iax_conf/",
]
+++

```
[general]
port=5036

;amaflags=default
;accountcode=lss0101
musiconhold=default
allow=all
allow=ulaw
allow=alaw
allow=gsm
bandwidth=low
jitterbuffer=no


register => username:password@212.115.0.0
register => udername:password@parcelfarce.domain.net	
register => username:password@iaxtel.com				


tos=lowdelay

; Guest section for unauthenticated connections

[guest]
type=user
context=inbound-from-iax
callerid="Guest IAX User"

[iaxtel]
type=user
context=inbound-from-iax
auth=rsa
inkeys=iaxtel

[iaxtel2]
type=user
context=inbound-from-iax
deny=0.0.0.0/0.0.0.0
permit=216.207.245.47/255.255.255.255

[parcelfarce]	;connection to parcelfarce
type=friend
auth=md5
secret=password
context=inbound-from-parcelfarce
host=212.116.0.0
qualify=yes
;trunk=yes

[ll]	;connection to ll
type=friend
host=212.115.0.0
username=ll
secret=password
context=inbound-from-iax
```