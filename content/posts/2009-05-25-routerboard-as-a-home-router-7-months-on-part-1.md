+++
title = "RouterBoard as a Home Router - 7 Months on Part 1"
#description = ""
date = "2009-06-19T12:06:33+01:00"
#updated = ""
#slug = ""
weight = 0
draft = false
#path = 
aliases = [
    "/?p=14",
    "/2009/05/25/routerboard-as-a-home-router-7-months-on-part-1/"
]
in_search_index = true

tags = [
    "Linux",
    "Mikrotik",
    "Router",
    "RouterBoard",
    "RouterOS",
    "Projects",
    "Hardware"
]
[extra]
+++
At the new year I decided that I was fed up with having my main Unix server acting as a Router (amongst other things) and decided to bite the bullet and get a full blown router. Here in lay a dilema. Being the fact that I’m a geek, I couldn’t settle for a “home” unhackable router. So this instantly ruled out most of the commercial available routers, baring those that run OpenWRT. Now don’t get me wrong, OpenWRT is more than capable, but I just didn’t feel like having to worry about hardware support, fighting with IPTables and getting hardware that probally wouldn’t scale. Now before anyone starts thinking “Scaling, but this is for a home connection!”, this is true. However I do sync my DSL at the full 24244 kbps Downstream, and 2550 kbps upstream (I live under 200m from the exchange according to my line attenuation, also my ISP doesn’t bandwidth cap, and allow for FastPath and similar to be enabled. Go BeThere!) . Also at the time, I was seriously considering investing in a secondary connection for additional bandwidth. This meant that I was left with a few choices

- Build my Own. Using something like an ALIX/Sokeris and use something like FreeBSD (or something with a webgui for when I feel rather lazy, such as m0n0wall or pfsense. Both I’ve used previously with great success)
- Cisco. Yes, the 800 pound gorrila of home. A ‘cheap’ 1800 or similar was going to set me back about £400, however this would have provided me most of what I needed.
- RotuerBoard. These where, to me at least, relativly unknown. I originally looked at them for building my own system with them, and then discovered RouterOS came with the boards. This was an instant sale.

After my first look at RouterOS I was basically sold. Main reasoning behind this was that it was a comercial Linux distribution, that actually worked well as a router, and shipped with both a CLI (Nortel-esq in this case) and a \*shock\* gui application. It also met my main criteria.

- Support for 802.1Q. I have multiple vLANs at home so having support for dot1q was a necessity
- Support for 802.3ad. As I have a few machines connecting via the router, I needed the throughput, as I don’t have gigabit switching LACP support was a necessity.
- Support for Wireless. All good routers for the home (even a geeky one) need support for 802.11(a/b/g).
- Support for SubSSIDs. Relating to the above, I didn’t want to have 7 wireless cards for my various networks
- Support for WPA2-PSK and WPA2-EAP. I use RADIUS to authenticate all my personal stations to a central authentication system, but I don’t want to have to add guests to this, so PSK should also be supported.
- Support for OpenVPN. I don’t like having my traffic to / from home going in the clear at all, so I needed to be able to connect via a VPN of some sort, My preference is OpenVPN for c2s vpns (s2s is still IPSEC…. which leads onto the next point)
- Support for IPSec. I connect to various friends networks, and yet again, don’t want this sort of traffic in the clear, we made the standard IPSec (3des/md5) a while back
- Support for “Unlimted” Firewall rules. This may sound silly, but anyone who has worked with the lowend Sonicwalls will know what I mean, only being able to put 20 rules is EXTREMELY restrictive especially with multiple vlans! (I’ve got roughly 300 rules)
- Support for setting DHCP options. I used VMWare ESX at home for my test lab, so I require to be able to setup the DHCP server to be able to send the correct options for PXE (or gPXE) so this was a requirement
- Quick booting. As silly as this may sound, I don’t want boot times of upwards of 30 seconds for my router.
- Support for Bridging of interfaces with Firewall rules. This one is rather self explanatory really!
- Support for UPnP. Lets face it, UPnP is required for any form of Voice/Video chat these days over the main IM networks (YIM/AIM/MSNIM)
- Support for NetFlow or similar. This one is a nice to have, as I like to use flow-tools to generate a rough guess on what type of traffic is flowing through my network
- Support for Traffic Shaping. Ah yes, the holy grail of routers. Unfortunately the likes of TC on linux requires a degree in astrophysics to get working how you’d like!
- Easy configuration.

After discovering (via the x86 installable and the demo units) that RouterOS would let me do all of the above, I decided to give it a whirl.