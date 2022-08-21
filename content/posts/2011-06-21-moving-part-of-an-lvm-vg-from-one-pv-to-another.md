+++
title = "Moving part of an lvm vg from one pv to another"
#description = ""
date = "2011-06-21T19:57:00+01:00"
#updated = ""
#slug = ""
weight = 0
draft = false
#path = 
aliases = [
    "/?p=231",
    "/2011/06/21/moving-part-of-an-lvm-vg-from-one-pv-to-another/"
    
]
in_search_index = true


tags = [
    "Linux",
    "Hardware"
]
[extra]
+++

 Lets say that you’ve got multiple physical volumes (PV) in a Volume Group (VG) and you want to migrate the extents from one PV to another, this can be acomplished with a quick and easy pvmove command.

For example  
```
pvdisplay -m
--- Physical volume ---
  PV Name               /dev/sdb1
  VG Name               INTEL_RAID
  PV Size               2.73 TiB / not usable 4.00 MiB
  Allocatable           yes (but full)
  PE Size               4.00 MiB
  Total PE              714539
  Free PE               0
  Allocated PE          714539
  PV UUID               XWiRzE-Ol3d-38En-ND6b-qo93-4zeF-xv8zDv

--- Physical Segments ---  
 Physical extent 0 to 604876:  
 Logical volume /dev/INTEL_RAID/MEDIA  
 Logical extents 0 to 604876  
 Physical extent 604877 to 617676:  
 Logical volume /dev/INTEL_RAID/backups_mimage_0  
 Logical extents 25600 to 38399  
 Physical extent 617677 to 617701:  
 Logical volume /dev/INTEL_RAID/EPG  
 Logical extents 0 to 24  
 Physical extent 617702 to 643301:  
 Logical volume /dev/INTEL_RAID/backups_mimage_0  
 Logical extents 0 to 25599  
 Physical extent 643302 to 714538:  
 Logical volume /dev/INTEL_RAID/MEDIA  
 Logical extents 604877 to 676113

--- Physical volume ---  
 PV Name /dev/sdc1  
 VG Name INTEL_RAID  
 PV Size 2.04 TiB / not usable 2.00 MiB  
 Allocatable yes  
 PE Size 4.00 MiB  
 Total PE 535726  
 Free PE 430323  
 Allocated PE 105403  
 PV UUID laOuKy-5FZa-cJ3h-JffV-qUub-diKC-O0wVqK

--- Physical Segments ---  
 Physical extent 0 to 25599:  
 Logical volume /dev/INTEL_RAID/backups_mimage_1  
 Logical extents 0 to 25599  
 Physical extent 25600 to 54202:  
 Logical volume /dev/INTEL_RAID/MEDIA  
 Logical extents 676114 to 704716  
 Physical extent 54203 to 67002:  
 Logical volume /dev/INTEL_RAID/NZB_DOWNLOAD  
 Logical extents 0 to 12799  
 Physical extent 67003 to 79802:  
 Logical volume /dev/INTEL_RAID/backups_mimage_1  
 Logical extents 25600 to 38399  
 Physical extent 79803 to 105402:  
 Logical volume /dev/INTEL_RAID/OLD_VM  
 Logical extents 0 to 25599  
 Physical extent 105403 to 535725:  
 FREE  

```  
From here you can see that /dev/INTEL_RAID/MEDIA is a Logical Volume (LV) on both PVs within the VG. If I was wanting to grow my mirrored LV, which requires space on both PVs, I’d have to migrate some of the extents of another LV. If I wanted to move some of the MEDIA lv, I should be able to do the following  
`pvmove /dev/sdb:643302-714538 /dev/sdc`  
This will move extents 643302-714538 to the next contiguious block on /dev/sdc