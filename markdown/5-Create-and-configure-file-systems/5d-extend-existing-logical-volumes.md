# 5.c Extend existing logical volumes

**This topic includes:**
+ Resizing logical volumes
+ Resizing filesystem
  + xfs
  + ext4

**Commands:**
+ lvextend (8)         - Add space to a logical volume
+ lvresize (8)           - Resize a logical volume
+ lvreduce (8)         - Reduce the size of a logical volume


## Logical Volumes

### Extending a Logical Volume

`lvextend` adds space to a logical volume. The space needs to be available in the volume group.  

When extending Logical Volumes, you do not need to unmount the partition (however you will need to extend the file system afterwards, or if the filesystems supports, use the '-r' flag to automatically resize the filesystem).  

Checking for available space

Use `vgs` to see the available space of the volume group

    # vgs vg1
      VG  #PV #LV #SN Attr   VSize  VFree  
      vg1   3   1   0 wz--n- <8.99g <4.99g
                               |      |- Available VG space (not allocated to a LV)
                               |- Total size of VG

You can use `lvs` to confirm that the LV is using the difference of the previous values

    # lvs vg1
      LV   VG  Attr       LSize Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
      lv1  vg1 -wi-a----- 4.00g  

Or just use `vgdisplay` and check the PE sizes

    # vgdisplay vg1 | grep 'PE /'
      Alloc PE / Size       1024 / 4.00 GiB
      Free  PE / Size       1277 / <4.99 GiB

### Extending the Logical Volume

Extend volume to specified size (k/m/g)

    # lvextend -L6G /dev/vg1/lv1
      Size of logical volume vg1/lv1 changed from 5.39 GiB (1381 extents) to 6.00 GiB (1536 extents).
      Logical volume vg1/lv1 successfully resized.

Extend the volume by 1GB

    # lvextend -L+1G /dev/vg1/lv1
      Size of logical volume vg1/lv1 changed from 6.00 GiB (1536 extents) to 7.00 GiB (1792 extents).
      Logical volume vg1/lv1 successfully resized.

Extend for the full available space in the VG

    # lvextend -l +100%FREE /dev/vg1/lv1
      Size of logical volume vg1/lv1 changed from 7.00 GiB (1792 extents) to <8.99 GiB (2301 extents).
      Logical volume vg1/lv1 successfully resized.

**⚠️ WARNING:** _`lvextend -l +100%FREE /dev/vg1/lv1` (without the plus size) will not work_

Extend to the percentage of the VG (60% or 8.99 = 5.394)

    # lvextend -l 60%VG /dev/vg1/lv1
      Size of logical volume vg1/lv1 changed from 4.00 GiB (1024 extents) to 5.39 GiB (1381 extents).
      Logical volume vg1/lv1 successfully resized.

You can also use the 'PE' size  

    # lvextend -l +1740 /dev/RHCSA/pinehead  
      Size of logical volume RHCSA/pinehead changed from <3.20 GiB (818 extents) to 9.99 GiB (2558 extents).
      Logical volume RHCSA/pinehead successfully resized.

**📝 NOTE:** *The 'r' option will attempt to resize the filesystem (if possible)*

### Shrinking a Logical Volume

Be careful when reducing an LV's size, because data in the reduced area is lost. Ensure that any file system on the LV is resized before running lvreduce so that the removed extents are not in use by the file system.

You can use two commands to shrink a logical volume:  
+ `lvreduce` reduces the size of an LV. The freed logical extents are returned to the VG to be used by other LVs.
+ `lvresize` resizes  an  LV  in  the same way as `lvextend` and `lvreduce`.

Shrink a logical volume by 2GB

    # lvresize -L-2G /dev/vg1/lv1
      WARNING: Reducing active logical volume to <6.99 GiB.
      THIS MAY DESTROY YOUR DATA (filesystem etc.)
    Do you really want to reduce vg1/lv1? [y/n]: y
      Size of logical volume vg1/lv1 changed from <8.99 GiB (2301 extents) to <6.99 GiB (1789 extents).
      Logical volume vg1/lv1 successfully resized.

Shrink a logical volume to 30% of the volume group size

    # lvreduce -l 30%VG  /dev/vg1/lv1
      WARNING: Reducing active logical volume to <2.70 GiB.
      THIS MAY DESTROY YOUR DATA (filesystem etc.)
    Do you really want to reduce vg1/lv1? [y/n]: y
      Size of logical volume vg1/lv1 changed from <6.99 GiB (1789 extents) to <2.70 GiB (691 extents).
      Logical volume vg1/lv1 successfully resized.

## Filesystems

**Commands:**
+ xfs_growfs (8)       - expand an XFS filesystem
+ resize2fs (8)           - ext2/ext3/ext4 file system resizer

After extending the logical volume, if you did not use the `-r` option you will need to extend the filesystem manually.  

### Extending XFS Filesystems

Check the size of the logical volume with 'lvs' or 'fdisk'

    # vgs vg2
      VG  #PV #LV #SN Attr   VSize  VFree
      vg2   3   1   0 wz--n- <6.99g    0  

    # fdisk -l /dev/vg2/xfs
    Disk /dev/vg2/xfs: 7 GiB, 7503609856 bytes, 14655488 sectors
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes

Compare it to the filesystem size with 'df'

    # df -hT /mnt/xfs
    Filesystem          Type  Size  Used Avail Use% Mounted on
    /dev/mapper/vg2-xfs xfs   6.0G   75M  6.0G   2% /mnt/xfs

Resize the filesystem with 'xfs_growfs' by giving the mount point

    # xfs_growfs /mnt/xfs
    meta-data=/dev/mapper/vg2-xfs    isize=512    agcount=4, agsize=392704 blks
             =                       sectsz=512   attr=2, projid32bit=1
             =                       crc=1        finobt=1, sparse=1, rmapbt=0
             =                       reflink=1
    data     =                       bsize=4096   blocks=1570816, imaxpct=25
             =                       sunit=0      swidth=0 blks
    naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
    log      =internal log           bsize=4096   blocks=2560, version=2
             =                       sectsz=512   sunit=0 blks, lazy-count=1
    realtime =none                   extsz=4096   blocks=0, rtextents=0
    data blocks changed from 1570816 to 1831936

Confirm that it has changed  

    # df -hT /mnt/xfs
    Filesystem          Type  Size  Used Avail Use% Mounted on
    /dev/mapper/vg2-xfs xfs   7.0G   83M  6.9G   2% /mnt/xfs

### Extending EXT4 Filesystems

Check the size of the logical volume with 'lvs' or 'fdisk'

    # lvs vg1
      LV   VG  Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
      ext4 vg1 -wi-ao---- <6.99g

    # fdisk -l /dev/vg1/ext4
    Disk /dev/vg1/ext4: 7 GiB, 7503609856 bytes, 14655488 sectors
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes

Compare it to the filesystem size with 'df'

    # df -hT /mnt/ext4/
    Filesystem           Type  Size  Used Avail Use% Mounted on
    /dev/mapper/vg1-ext4 ext4  5.9G   24M  5.5G   1% /mnt/ext4

Resize the filesystem with 'resize2fs' by giving the device name

    # resize2fs /dev/mapper/vg1-ext4
    resize2fs 1.45.6 (20-Mar-2020)
    Filesystem at /dev/mapper/vg1-ext4 is mounted on /mnt/ext4; on-line resizing required
    old_desc_blocks = 1, new_desc_blocks = 1
    The filesystem on /dev/mapper/vg1-ext4 is now 1831936 (4k) blocks long.

Confirm that it has changed  

    # df -hT /mnt/ext4
    Filesystem           Type  Size  Used Avail Use% Mounted on
    /dev/mapper/vg1-ext4 ext4  6.9G   27M  6.5G   1% /mnt/ext4

---
[⬅️ Back](5-Create-and-configure-file-systems.md)
