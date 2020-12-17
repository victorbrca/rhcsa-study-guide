# 4.c Assign physical volumes to volume groups

## Working with Volume Groups

**Commands:**
+ lvm (8)                  - LVM2 tools
+ vgcreate (8)         - Create a volume group
+ vgdisplay (8)        - Display volume group information
+ vgextend (8)        - Add physical volumes to a volume group
+ vgreduce (8)        - Remove physical volume(s) from a volume group
+ vgremove (8)      - Remove volume group(s)
+ vgs (8)                  - Display information about volume groups

### Creating a Volume Group

`vgcreate` creates a new VG on block devices. If the devices were not previously initialized as PVs with `pvcreate`, `vgcreate` will initialize them, making them PVs. The `pvcreate` options for initializing devices are also available with `vgcreate`.

We create a volume group with `vgcreate` (Create a volume group)

    # vgcreate [vg name] /dev/device /dev/device2 /dev/device3

For example

    # vgcreate vg1 /dev/sdb /dev/sdc
      Volume group "vg1" successfully created

Listing the new volume group (with `vgs [volume group]`)

    # vgs vg1
      VG   #PV #LV #SN Attr   VSize   VFree
      vg1    2   0   0 wz--n-   5.99g 5.99g

Listing all volume groups (`vgs`)

    # vgs
      VG   #PV #LV #SN Attr   VSize   VFree
      rhel   1   2   0 wz--n- <29.00g    0  
      vg1    2   0   0 wz--n-   5.99g 5.99g

Or with more details

    # vgdisplay vg1
      --- Volume group ---
      VG Name               vg1
      System ID              
      Format                lvm2
      Metadata Areas        2
      Metadata Sequence No  1
      VG Access             read/write
      VG Status             resizable
      MAX LV                0
      Cur LV                0
      Open LV               0
      Max PV                0
      Cur PV                2
      Act PV                2
      VG Size               5.99 GiB
      PE Size               4.00 MiB
      Total PE              1534
      Alloc PE / Size       0 / 0    
      Free  PE / Size       1534 / 5.99 GiB
      VG UUID               uvHpRZ-BdPH-Nzxy-Lp15-VMps-fzPZ-A1bebc

Remember, you can also create a PV with `vgcreate` (bypassing the need to run `pvcreate`)

    # vgcreate vg2 /dev/sdd
      Physical volume "/dev/sdd" successfully created.
      Volume group "vg2" successfully created

### Extending a Volume Group

You can use `vgextend` to extend volume groups by adding physical volumes to it.

Initialize the new drive as a physical volume

    # pvcreate /dev/sde
      Physical volume "/dev/sde" successfully created.

Add the new physical volume to the volume group

    # vgextend vg1 /dev/sde
      Volume group "vg1" successfully extended

### Reducing a Volume Group

`vgreduce` removes one or more unused PVs from a VG.

Let's look at our volume group. Note it has 8.99GB of space

    # vgs vg1
      VG  #PV #LV #SN Attr   VSize  VFree  
      vg1   3   0   0 wz--n- <8.99g <8.99g

Remove one of the physical volumes

    # vgreduce vg1 /dev/sde
      Removed "/dev/sde" from volume group "vg1"

List the volume group again and now it has 5.99GB

    # vgs vg1
      VG  #PV #LV #SN Attr   VSize VFree
      vg1   2   0   0 wz--n- 5.99g 5.99g

### Deleting/Removing a Volume Group

`vgremove` removes one or more VGs. If LVs exist in the VG, a prompt is used to confirm LV removal.

    # vgremove vg1
      Volume group "vg1" successfully removed

---
[⬅️ Back](4-Configure-local-storage.md)
