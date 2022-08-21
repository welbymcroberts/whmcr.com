+++
title = "iSCSI Target re scanning on VMWare"
#description = ""
date = "2013-05-30T13:46:20+01:00"
#updated = ""
#slug = ""
weight = 0
draft = false
#path = 
aliases = [
    "/?p=287",
    "/2013/05/30/iscsi-target-re-scanning-on-vmware/"
]
in_search_index = true

tags = [
    "esx", "vmware", "Hardware",
]
+++

If you’re using iSCSI on VMWare but have a requirement to rescan the luns after a machine has booted (For example a VM which has DirectPath to a Storage card enabled, which is hosting your iSCSI luns) you can simply do so with the following command

```bash
ssh USER@ESXIHOST-- 'esxcli storage core adapter rescan --all && esxcfg-rescan -A' 1>/dev/null 2>/dev/null
```