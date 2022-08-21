+++
title = "Issues with OS X 10.5 iTunes 8.1.1 and mt-daapd (aka Firefly Media Server)"
#description = ""
date = "2009-05-25T19:01:50+01:00"
#updated = ""
#slug = ""
weight = 0
draft = false
#path = 
aliases = [
    "/?p=9",
    "/2009/05/25/issues-with-os-x-105-itunes-811-and-mt-daapd-aka-firefly-media-server/"
]
in_search_index = true

tags = [
    "Avahi",
    "Bonjour",
    "Bug",
    "iTunes",
    "Linux",
    "Mac",
    "Macintosh",
    "mt-daapd",
    "OSX",
    "Rendezvous",
    "Apple",
    "iTunes",
    "Linux",
    "media",
    "Software",
]
[extra]
+++
I’ve recently upgraded my iTunes installation on my MacBookPro to 8.1.1 and to my horror found that I’m no longer able to connect to my DAAP library on my NAS.

This is rather strange as the issue has only just appeared in 8.1.1 and does not appear on my windows machines which reside on a different network, and have Bonjour / Rendezvous mDNS traffic broadcast locally by [RendevousProxy](http://ileech.sourceforge.net/index.php?content=RendezvousProxy-News). After much annoyance, I decided to do a quick check of what an older iTunes library was sending out, and comparing that to Avahi. It turns out that my Avahi configuration was missing some vital Text Records. This wasn’t an issue in previous revisions of the iTunes client, but appears to be an issue in 8.1.1.

I updated my `daap.service` file in `/etc/avahi/services/` to the following

```
%h
      _daap._tcp
          3689
	  txtvers=1
	  iTSh Version=131073
	  Version=196610  

```

And restarted Avahi for good measure and now can connect to my mt-daapd library again!