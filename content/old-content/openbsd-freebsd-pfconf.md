+++
title = 'OpenBSD / FreeBSD pf.conf'
date = '2005-05-25T20:55:17+01:00'
guid = 'http://www.whmcr.com/?page_id=39'
aliases = [
    "/?p=27",
    "/2005/05/25/openbsd-freebsd-pfconf/",
    "/pages/8/",
    "/old-content/openbsd-freebsd-pfconf/",
    "/openbsd-freebsd-pfconf/"
]
url = "/openbsd-freebsd-pfconf"
+++


Here is a copy of my simple home pf.conf. I run this on a freebsd 5.3 box.  
ed0 is pluged into my dsl modem, rl0 internally

```


$ cat /etc/pf.conf
# Macros
ext_if="ed0"
int_if="rl0"
internal_net="192.168.0.0/24"
external_addr="80.xx.xx.xx"

set block-policy drop
set require-order yes
set fingerprints "/etc/pf.os"
set loginterface $ext_if

#scrub adubdub
scrub in all

nat on $ext_if from $internal_net to any -> ($ext_if)
#ftp
rdr on $ext_if proto tcp from any to $external_addr/32 port 21 -> 192.168.0.2 port 21
#transparant proxy, block ie
rdr on $int_if inet proto tcp from any to ! 192.168.0.1  port 80 -> 127.0.0.1 port 3128
#iax(2)
rdr on $ext_if proto udp from any to $external_addr/32 port 4569 -> 192.168.0.5 port 4569


# Filtering: the implicit first two rules are
pass in all
pass out all

# block all incoming packets but allow ssh, pass all outgoing tcp and udp
# connections and keep state, logging blocked packets.
block in log all

#nmap
block in log quick on $ext_if inet proto tcp from any to any flags FUP/FUP

#iax2
pass in on $ext_if proto udp from any to any port 4569 keep state
#http
pass in on $ext_if proto tcp from any to any port 80 keep state
#ftp
pass in on $ext_if proto tcp from any to any port 21 keep state
#icmp, ping etc
pass in on $ext_if proto icmp all




#allow outbound
#anything really
pass out on $ext_if proto { tcp, udp, icmp } all keep state





#open everything on internal ... if you don't trust that side of the network, you've got big probs
pass in on $int_if all
```