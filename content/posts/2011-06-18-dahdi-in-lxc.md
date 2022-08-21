+++
title = "Dahdi In LXC"
#description = ""
date = "2011-06-18T19:50:20+01:00"
#updated = ""
#slug = ""
weight = 0
draft = false
#path = 
aliases = [
    "/?p=191",
    "/2011/06/18/dahdi-in-lxc/"
]
in_search_index = true


tags = [
    "lxc",
    "asterisk",
    "voip",
    "freetdm",
    "dahdi",
    "freeswitch",
    "Linux",
    "Telephony",
    "Containers"
]
[extra]
+++

At home we use various VoIP providers to either get free calls to various places (GTalk/GVoice to america for instance) and various other destinations over SIP providers

I’ve been using Asterisk for years (I remeber the 0.7 release) and have implemented it for companies before, usually with no issues, baring the continuall deadlocks in the 1.2 range. Recently I enabled my VoIP network segment for IPv6 only to find that GTalk stoped working on IPV6 Day. After a bit of digging about, it seems that Asterisk 1.8 does support IPV6! But, gtalk and similar are not supported, SIP is infact the only first class citezen it seems.

I’ve toyed with using freeswitch before, but unfortuantly have had varied sucsess with FreeTDM to Dahdi with BT caller ID and the likes. I did hack in support for it, but I’m not too sure if I trust my code, as my C is quite rusty to say the least.

I did however come up with another solution!

As I’m running a moderatly new Linux Kernel I can use LXC – Linux Containers – which are effectilvy the same idea as a wpar, chroot, openvz, whatever. After setting up asterisk in the LXC I needed to expose my Dahdi card to it. LXC allows you to restrict access on a per device basis. I’ve setup Dahdi on the host machine as normal so the kernel modules can be loaded etc. Once this is done I’ve preformed the following within the LXC  
```
cd /
mkdir dev/dahdi
mknod dev/dahdi/1 c 196 1
mknod dev/dahdi/2 c 196 2
mknod dev/dahdi/3 c 196 3
mknod dev/dahdi/4 c 196 4
mknod dev/dahdi/channel c 196 254
mknod dev/dahdi/ctl c 196 0
mknod dev/dahdi/pseudo c 196 255
mknod dev/dahdi/timer c 196 253
mknod dev/dahdi/transcode c 196 250
```
This creates the Device nodes within `/dev/` for my 4 dahdi channels (3FXS 1FXO if anyone is interested). After this I’ve added the following to the lxc config file, to allow the LXC to have access to these devices

```
# If you want to be lazy just add this line
#lxc.cgroup.devices.allow = c 196:* rwm`

#Otherwise use the following  
lxc.cgroup.devices.allow = c 196:0 rwm  
lxc.cgroup.devices.allow = c 196:1 rwm  
lxc.cgroup.devices.allow = c 196:2 rwm  
lxc.cgroup.devices.allow = c 196:3 rwm  
lxc.cgroup.devices.allow = c 196:4 rwm  
lxc.cgroup.devices.allow = c 196:250 rwm  
lxc.cgroup.devices.allow = c 196:253 rwm  
lxc.cgroup.devices.allow = c 196:254 rwm  
lxc.cgroup.devices.allow = c 196:255 rwm  
```

This will obviously only work for the first 4 dahdi channels, but if you need more, just continue adding the 196:x lines, replacing x with the channel number, and also ensuring that you create the device nodes in the same way