+++
date = "2022-08-16T20:00:00+00:00"
title = "LLDP on Intel X710 Network Cards"
#description = ""
#updated = ""
#slug = ""
#url = ""
aliases = []

tags = [
    "Infrastructure",
    "Networking",
    "lldp",
    "Linux",
    "Networking"
]
+++

It's sometimes useful to be able to match ports on a system to those on a switch from both sides. Security concerns aside for a moment, this can be done using one of the many link layer protocols, CDP and LLDP being examples of this. On a linux system `lldpd` is a good lightweight utility that handles the LLDP PDUs from the switch, and sends out its own.

However, some network cards will make the frames sent 'disapear' and just make the ones receive disapear into the ether from the perspective of the OS.

Intel's X700 range of cards does this. It's part of the ['data centre bridging'](https://www.kernel.org/doc/html/v5.2/networking/device_drivers/intel/i40e.html#data-center-bridging-dcb) support that the card has, and whilst this may be useful for DCB/DCBx, in most situations this can cause a bit of pain. 

On older linux systems, using debugfs you can disable the LLDP Firmware Agent, however modern versions, or at least those with a >3.10 kernel, and a modern ethtool, can use the ethtool command to set a flag to be enabled, which disables the agent.

` ethtool --set-priv-flags INTERFACE disable-fw-lldp on `


You can confirm this is working using
``` ethtool --show-priv-flags INTERFACE ```

To set this on every boot, a tool like udev is likley to work best, in a udev rules file, for example, /etc/udev/rules.d/99-net-disable-fw-lldp.rules

```
# Intel x710's have a firmware LLDP agent, it sends incorrect LLDP packets, and stops the OS from receiving them. Disabling
ACTION=="add", SUBSYSTEM=="net", ENV{INTERFACE}=="*", DRIVERS=="i40e", PROGRAM="/usr/sbin/ethtool --set-priv-flags $name disable-fw-lldp on"
```

