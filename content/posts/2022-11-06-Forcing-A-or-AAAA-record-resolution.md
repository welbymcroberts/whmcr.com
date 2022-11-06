+++
date = "2022-11-06T12:45:00-05:00"
title = "Forcing A or AAAA record resolution with CoreDNS"
#description = ""
#updated = ""
#slug = ""
#url = ""
aliases = []

tags = [
    "Infrastructure",
    "Networking",
    "coredns",
    "Linux",
]
+++

Some devices will favour IPv6 when configured, which is great, however there are sometimes that you want a device to only resolve either the v4 or v6 addresses. In this case it was for ICMP monitoring of remote devices. The device doing the monitoring will favour IPv6 over IPv4, which is great, however I want to be able to monitor both.

[CoreDNS](https://coredns.io/) is the DNS server which I use for serving up internal DNS, so I was looking for a way to use this to perform the forcing of records.

I went back and forth on ways to do this, templates, different forwardings based on domain and so on, however I ended up using the [rewrite](https://coredns.io/plugins/rewrite/) to acheive this.

An example Corefile would be as follows (with my other configuration removed)


```
(bind) {
  bind 127.0.0.1
  bind 127.0.0.53
  bind ::1
  bind 192.168.1.2
}
a:53 {
  import bind
  rewrite continue { name regex (.*)\.a {1} answer auto }
  rewrite continue { type AAAA A }
  forward . 8.8.8.8
}
aaaa:53 {
  import bind
  rewrite continue { name regex (.*)\.aaaa {1} answer auto }
  rewrite continue { type A AAAA }
  forward . 8.8.8.8
}
.:53 {
  import bind
  forward . 1.1.1.1
}
```

This configuration listens on 3 different zones. 

1. The .:53 stanza will configure CoreDNS to forward any query received on one of the bound IPs (as per the bind template at the top of the file) to 1.1.1.1.

2. The a:53 stanza will reqrite any query that ends .a and will strip the .a from the end for the forwarded query, and re-add on the inbound. This will also rewrite CNAME records to be the 'correct' type. The next rewrite is to Force any AAAA requests to be rewritten to A requests. It then forwards on to 8.8.8.8.

3. The aaaa:53 stanza does the sames as #2 does, but for AAAA records.

By pushing queries through this, its possible to force devices that have no other way to force v4 / v6 resolution to resolve whichever is prefered.

* Testing an .A record

```
~ % dig icanhazip.com.a @192.168.1.2 -t AAAA

; <<>> DiG 9.10.6 <<>> icanhazip.com.a @192.168.1.2
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 32139
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;icanhazip.com.a.		IN	AAAA

;; ANSWER SECTION:
icanhazip.com.a.	92	IN	A	104.18.114.97
icanhazip.com.a.	92	IN	A	104.18.115.97

;; Query time: 19 msec
;; SERVER: 192.168.1.2#53(192.168.1.2)
;; WHEN: Sun Nov 06 12:42:08 EST 2022
;; MSG SIZE  rcvd: 106
```


* Testing an AAAA record being forced
```
~ % dig icanhazip.com.aaaa @192.168.1.2 -t A

; <<>> DiG 9.10.6 <<>> icanhazip.com.aaaa @192.168.1.2
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 28438
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;icanhazip.com.aaaa.		IN	A

;; ANSWER SECTION:
icanhazip.com.aaaa.	30	IN	AAAA	2606:4700::6812:7361
icanhazip.com.aaaa.	30	IN	AAAA	2606:4700::6812:7261

;; Query time: 97 msec
;; SERVER: 192.168.1.2#53(192.168.1.2)
;; WHEN: Sun Nov 06 12:42:13 EST 2022
;; MSG SIZE  rcvd: 139
```

* Testing without specifying
```
~ % dig icanhazip.com @192.168.1.2

; <<>> DiG 9.10.6 <<>> icanhazip.com @192.168.1.2
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 28423
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 13, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;icanhazip.com.			IN	A

;; ANSWER SECTION:
icanhazip.com.		21	IN	A	104.18.115.97
icanhazip.com.		21	IN	A	104.18.114.97

;; AUTHORITY SECTION:
.			42144	IN	NS	b.root-servers.net.
.			42144	IN	NS	i.root-servers.net.
.			42144	IN	NS	m.root-servers.net.
.			42144	IN	NS	h.root-servers.net.
.			42144	IN	NS	c.root-servers.net.
.			42144	IN	NS	k.root-servers.net.
.			42144	IN	NS	f.root-servers.net.
.			42144	IN	NS	g.root-servers.net.
.			42144	IN	NS	j.root-servers.net.
.			42144	IN	NS	e.root-servers.net.
.			42144	IN	NS	l.root-servers.net.
.			42144	IN	NS	d.root-servers.net.
.			42144	IN	NS	a.root-servers.net.

;; Query time: 8 msec
;; SERVER: 192.168.1.2#53(192.168.1.2)
;; WHEN: Sun Nov 06 12:42:17 EST 2022
;; MSG SIZE  rcvd: 503
```
