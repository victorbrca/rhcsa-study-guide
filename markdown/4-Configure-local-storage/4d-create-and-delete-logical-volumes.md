# 4.d Create and delete logical volumes

## Working with Logical Volumes

**Commands:**
+ lvm (8)                  - LVM2 tools
+ lvcreate (8)          - Create a logical volume
+ lvdisplay (8)         - Display information about a logical volume
+ lvremove (8)        - Remove logical volume(s) from the system
+ lvs (8)                    - Display information about logical volumes

### Creating a Logical Volume

    # lvcreate -L 4g [vg name] -n [lv name]

Flags:
+ `-n`  - set the Logical Volume name
+ `-l` - use extents rather than a specified size

**Example:**

Create the LV

    # lvcreate -L 4g vg1 -n lv1
      Logical volume "lv1" created.

Display simple information about the LV

    # lvs vg1
      LV   VG  Attr       LSize Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
      lv1  vg1 -wi-a----- 4.00g   

Simple information with verbose

    # lvs -v vg1
      LV   VG  #Seg Attr       LSize Maj Min KMaj KMin Pool Origin Data%  Meta%  Move Cpy%Sync Log Convert LV UUID                                LProfile
      lv1  vg1    2 -wi-a----- 4.00g  -1  -1  253    2                                                     ADUPcG-YAuo-5vDC-7FEB-Cas9-4Gt0-hR1kVD  

Detailed information

    # lvdisplay vg1
      --- Logical volume ---
      LV Path                /dev/vg1/lv1
      LV Name                lv1
      VG Name                vg1
      LV UUID                ADUPcG-YAuo-5vDC-7FEB-Cas9-4Gt0-hR1kVD
      LV Write Access        read/write
      LV Creation host, time localhost.localdomain, 2020-11-18 08:07:29 -0500
      LV Status              available
      # open                 0
      LV Size                4.00 GiB
      Current LE             1024
      Segments               2
      Allocation             inherit
      Read ahead sectors     auto
      - currently set to     8192
      Block device           253:2

### Deleting/Removing a Logical Volume

`lvremove`  removes one or more LVs. For standard LVs, this returns the logical extents that were used by the LV to the VG for use by other LVs.

    # lvremove /dev/vg1/lv1
    Do you really want to remove active logical volume vg1/lv1? [y/n]: y
      Logical volume "lv1" successfully removed

You can also specify the logical volume with `[vg name]/[lv name]`

    # lvremove vg1/lv1
    Do you really want to remove active logical volume vg1/lv1? [y/n]: y
      Logical volume "lv1" successfully removed

---
[⬅️ Back](4-Configure-local-storage.md)
