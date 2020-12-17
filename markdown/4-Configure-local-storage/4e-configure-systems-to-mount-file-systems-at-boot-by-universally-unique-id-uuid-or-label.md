# 4.e Configure systems to mount file systems at boot by Universally Unique ID (UUID) or label

## Non-persistent naming attributes

Traditionally, non-persistent names in the form of `/dev/sd*(major number)**(minor number)*` are used on Linux to refer to storage devices. The major and minor number range and associated `sd` names are allocated for each device when it is detected. This means that the association between the major and minor number range and associated `sd` names can change if the order of device detection changes.

Such a change in the ordering might occur in the following situations:

+ The parallelization of the system boot process detects storage devices in a different order with each system boot.
+ A disk fails to power up or respond to the SCSI controller.
+ A SCSI controller (host bus adapter, or HBA) fails to initialize, causing all disks connected to that HBA to not be detected.
+ The order of driver initialization changes if different types of HBAs are present in the system.
+ Disks connected to the system with Fibre Channel, iSCSI, or FCoE adapters might be inaccessible at the time the storage devices are probed, due to a storage array or intervening switch being powered off, for example.

These reasons make it undesirable to use the major and minor number range or the associated `sd` names when referring to devices, such as in the `/etc/fstab` file.

## Persistent naming attributes

### udev Naming Mechanism

The udev mechanism is used for all types of devices in Linux, not just for storage devices. In the case of storage devices, udev rules creates pemanent device identifiers via symbolic links in the `/dev/disk/` directory. This enables you to refer to storage devices by:

+ Their content
+ A unique identifier
+ Their serial number.

**📝 NOTE:** *Although `udev` naming attributes are persistent, in that they do not change on their own across system reboots, some are also configurable.*

### File system identifiers

**File system identifiers are tied to a particular file system** created on a block device. The identifier is also stored as part of the file system. If you copy the file system to a different device, it still carries the same file system identifier. On the other hand, if you rewrite the device, such as by formatting it with the `mkfs` utility, the device loses the attribute.

File system identifiers include:
+ Unique identifier (UUID)
+ Label

#### The UUID attribute in `/dev/disk/by-uuid/`

Entries in this directory provide a symbolic name that refers to the storage device by a **unique identifier** (UUID). You can use the UUID to refer to the device in the `/etc/fstab` file using the following syntax:

    UUID=3e6be9de-8139-11d1-9106-a43f08d823a6

You can configure the UUID attribute when creating a file system, and you can also change it later on.

#### The Label attribute in `/dev/disk/by-label/`

Entries in this directory provide a symbolic name that refers to the storage device by a **label**. You can use the label to refer to the device in the /etc/fstab file using the following syntax:

    LABEL=Boot

### Device identifiers

**📝 NOTE:** *Device identifiers should not be needed for the exam, but it's good knowledge to have*

Device identifiers are tied to a block device: for example, a disk or a partition. If you rewrite the device, such as by formatting it with the `mkfs` utility, the device keeps the attribute, because it is not stored in the file system.

Device identifiers include:

+ World Wide Identifier (WWID)
+ Partition UUID
+ Serial number

#### PARTUUID vs UUID

+ UUID is a filesystem-level UUID, which is retrieved from the filesystem metadata inside the partition. That can only be read if the filesystem type is known and readable.
+ PARTUUID is a partition-table-level UUID for the partition, a standard feature for all partitions on GPT-partitioned disks. Since it is retrieved from the partition table, it is accessible without making no assumptions at all about the actual contents of the partition. If the partition is encrypted using some unknown encryption method, this might be the only accessible unique identifier for that particular partition.

Both 'UUID' and 'PARTUUID' can be used in `/etc/fstab`.

## Getting the file system identifiers

### Getting UUID

There are different ways to get the UUID for a partition.

Using `blkid`

    # blkid
    /dev/sda1: UUID="375fcafb-dbf0-4e72-a1de-9bba811fc6d4" BLOCK_SIZE="512" TYPE="xfs" PARTUUID="f5953a5f-01"
    /dev/sda2: UUID="fv8aQX-tfJD-3ieh-G2wo-GrnR-bcML-j41Dis" TYPE="LVM2_member" PARTUUID="f5953a5f-02"
    /dev/mapper/rhel-root: UUID="cc50014d-ed1f-4785-a5f2-304ec6da9002" BLOCK_SIZE="512" TYPE="xfs"
    /dev/mapper/rhel-swap: UUID="01d46e11-4725-4743-8d92-ca9fdd675a8e" TYPE="swap"

Using `lsblk -f`

    # lsblk -f
    NAME          FSTYPE      LABEL UUID                                   MOUNTPOINT
    sda
    ├─sda1        xfs               375fcafb-dbf0-4e72-a1de-9bba811fc6d4   /boot
    └─sda2        LVM2_member       fv8aQX-tfJD-3ieh-G2wo-GrnR-bcML-j41Dis
      ├─rhel-root xfs               cc50014d-ed1f-4785-a5f2-304ec6da9002   /
      └─rhel-swap swap              01d46e11-4725-4743-8d92-ca9fdd675a8e   [SWAP]

Using `ls /dev/disk/by-uuid/`

    # ll /dev/disk/by-uuid/
    total 0
    lrwxrwxrwx. 1 root root 10 Nov 17 16:48 01d46e11-4725-4743-8d92-ca9fdd675a8e -> ../../dm-1
    lrwxrwxrwx. 1 root root 10 Nov 17 16:48 375fcafb-dbf0-4e72-a1de-9bba811fc6d4 -> ../../sda1
    lrwxrwxrwx. 1 root root 10 Nov 17 16:48 cc50014d-ed1f-4785-a5f2-304ec6da9002 -> ../../dm-0

### Getting UUID for LVM

You can also use `blkid` or `lsblkd -f` to get the UUID for logical volumes.

    # pvs
      PV         VG   Fmt  Attr PSize   PFree
      /dev/sda2  rhel lvm2 a--  <29.00g       0
      /dev/sdb   vg1  lvm2 a--   <3.00g       0
      /dev/sdc   vg1  lvm2 a--   <3.00g 1016.00m
      /dev/sdd        lvm2 ---    3.00g    3.00g
      /dev/sde   vg1  lvm2 a--   <3.00g   <3.00g

With `blkid`

    # blkid
    /dev/sda1: UUID="375fcafb-dbf0-4e72-a1de-9bba811fc6d4" BLOCK_SIZE="512" TYPE="xfs" PARTUUID="f5953a5f-01"
    /dev/sda2: UUID="fv8aQX-tfJD-3ieh-G2wo-GrnR-bcML-j41Dis" TYPE="LVM2_member" PARTUUID="f5953a5f-02"
    /dev/mapper/rhel-root: UUID="cc50014d-ed1f-4785-a5f2-304ec6da9002" BLOCK_SIZE="512" TYPE="xfs"
    /dev/mapper/rhel-swap: UUID="01d46e11-4725-4743-8d92-ca9fdd675a8e" TYPE="swap"
    /dev/sdc: UUID="qIVN0M-bpnS-2nDK-SuTE-l3bD-8Y3d-52kVwz" TYPE="LVM2_member"
    /dev/sdb: UUID="8vojd0-AgE9-fDqa-pVWq-X0sl-29ll-OChdzL" TYPE="LVM2_member"
    /dev/sdd: UUID="8H5qa7-7UiT-ydRL-Jc50-BSdv-weoj-4TCVJo" TYPE="LVM2_member"
    /dev/sde: UUID="WwyG65-4kn1-GKHN-pgW5-X2Os-UAFc-Ca2rfB" TYPE="LVM2_member"
    /dev/mapper/vg1-lv1: LABEL="lv1" UUID="83435299-3439-4db6-a7fd-ba86bbdb2387" BLOCK_SIZE="4096" TYPE="ext4"

Specifying the LV path

    # blkid /dev/mapper/vg1-lv1
    /dev/mapper/vg1-lv1: LABEL="lv1" UUID="83435299-3439-4db6-a7fd-ba86bbdb2387" BLOCK_SIZE="4096" TYPE="ext4"

With `lsblk`

    # lsblk  -f
    NAME          FSTYPE      LABEL UUID                                   MOUNTPOINT
    sda
    ├─sda1        xfs               375fcafb-dbf0-4e72-a1de-9bba811fc6d4   /boot
    └─sda2        LVM2_member       fv8aQX-tfJD-3ieh-G2wo-GrnR-bcML-j41Dis
      ├─rhel-root xfs               cc50014d-ed1f-4785-a5f2-304ec6da9002   /
      └─rhel-swap swap              01d46e11-4725-4743-8d92-ca9fdd675a8e   [SWAP]
    sdb           LVM2_member       8vojd0-AgE9-fDqa-pVWq-X0sl-29ll-OChdzL
    └─vg1-lv1     ext4        lv1   83435299-3439-4db6-a7fd-ba86bbdb2387
    sdc           LVM2_member       qIVN0M-bpnS-2nDK-SuTE-l3bD-8Y3d-52kVwz
    └─vg1-lv1     ext4        lv1   83435299-3439-4db6-a7fd-ba86bbdb2387
    sdd           LVM2_member       8H5qa7-7UiT-ydRL-Jc50-BSdv-weoj-4TCVJo
    sde           LVM2_member       WwyG65-4kn1-GKHN-pgW5-X2Os-UAFc-Ca2rfB

### Getting the Label

Like with UUID, you can get the label with `lsblk -f`

    # lsblk -f
    NAME          FSTYPE      LABEL  UUID                                   MOUNTPOINT
    sda                                                                     
    ├─sda1        xfs                375fcafb-dbf0-4e72-a1de-9bba811fc6d4   /boot
    └─sda2        LVM2_member        fv8aQX-tfJD-3ieh-G2wo-GrnR-bcML-j41Dis
      ├─rhel-root xfs                cc50014d-ed1f-4785-a5f2-304ec6da9002   /
      └─rhel-swap swap               01d46e11-4725-4743-8d92-ca9fdd675a8e   [SWAP]
    sdb           LVM2_member        lJvOPK-AyOu-czH5-c96Y-xsmL-2Vnn-prLgpN
    └─vg1-lv1                                                               
    sdc           LVM2_member        lAzsiO-trcw-BbGl-nbQe-bLo6-h4pr-wL09wa
    └─vg1-lv1                                                               
    sdd                                                                     
    └─sdd1        ext4        ext4fs cae6fca6-e7b4-45b4-93ea-23430424ed7a   


Or with `lsblk --output +LABEL`

    # lsblk --output +LABEL
    NAME          MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT LABEL
    sda             8:0    0   30G  0 disk            
    ├─sda1          8:1    0    1G  0 part /boot      
    └─sda2          8:2    0   29G  0 part            
      ├─rhel-root 253:0    0   27G  0 lvm  /          
      └─rhel-swap 253:1    0  2.1G  0 lvm  [SWAP]     
    sdb             8:16   0    3G  0 disk            
    └─vg1-lv1     253:2    0    4G  0 lvm             
    sdc             8:32   0    3G  0 disk            
    └─vg1-lv1     253:2    0    4G  0 lvm             
    sdd             8:48   0    3G  0 disk            
    └─sdd1          8:49   0    3G  0 part            ext4fs

With `blkid`

    # blkid
    /dev/sdd1: LABEL="ext4fs" UUID="cae6fca6-e7b4-45b4-93ea-23430424ed7a" BLOCK_SIZE="4096" TYPE="ext4" PARTLABEL="Linux filesystem" PARTUUID="a866a6e2-30be-4f5e-86e7-b5ee2830467f"
    /dev/sdc: UUID="lAzsiO-trcw-BbGl-nbQe-bLo6-h4pr-wL09wa" TYPE="LVM2_member"
    /dev/sdb: UUID="lJvOPK-AyOu-czH5-c96Y-xsmL-2Vnn-prLgpN" TYPE="LVM2_member"
    /dev/sda1: UUID="375fcafb-dbf0-4e72-a1de-9bba811fc6d4" BLOCK_SIZE="512" TYPE="xfs" PARTUUID="f5953a5f-01"
    /dev/sda2: UUID="fv8aQX-tfJD-3ieh-G2wo-GrnR-bcML-j41Dis" TYPE="LVM2_member" PARTUUID="f5953a5f-02"
    /dev/mapper/rhel-root: UUID="cc50014d-ed1f-4785-a5f2-304ec6da9002" BLOCK_SIZE="512" TYPE="xfs"
    /dev/mapper/rhel-swap: UUID="01d46e11-4725-4743-8d92-ca9fdd675a8e" TYPE="swap"

And with `ll /dev/disk/by-label/`

    # ll /dev/disk/by-label/
    total 0
    lrwxrwxrwx. 1 root root 10 Dec 13 17:15 ext4fs -> ../../sdd1


## Configuring the Mount Point

See [5.a Create, mount, unmount, and use vfat, ext4, and xfs file systems](../5-Create-and-configure-file-systems/5a-create-mount-unmount-and-use-vfat-ext4-and-xfs-file-systems.md) for information on the 'fstab' and mounting filesystems.

### Mounting LVM

Proceed as any other partition.

After formating the logical volume, get the UUID

    # blkid /dev/mapper/vg1-lv1
    /dev/mapper/vg1-lv1: LABEL="lv1" UUID="83435299-3439-4db6-a7fd-ba86bbdb2387" BLOCK_SIZE="4096" TYPE="ext4"

Create the mount point

    # mkdir /mnt/lv1

Add it to `/etc/fstb`

    # LV Mount
    UUID=83435299-3439-4db6-a7fd-ba86bbdb2387    /mnt/lv1    ext4     defaults    0 2

Test the mount

    # mount -a

Check that it worked

    # df /mnt/lv1/
    Filesystem          1K-blocks  Used Available Use% Mounted on
    /dev/mapper/vg1-lv1   5095040 20472   4796040   1% /mnt/lv1

**📝 NOTE:** *While you can safely use the device mapper path for LVMs, for the purposed of this objective we used the UUID.*

---

## Changing FS Labels

This is good knowledge to know for the exam.

### For EXT4

**Commands:**
+ e2label - Change the label on an ext2/ext3/ext4 filesystem
+ tune2fs - adjust tunable filesystem parameters on ext2/ext3/ext4 filesystems

With `e2label`

    # e2label /dev/mapper/vg1-lv1 logical-volume1

With `tune2fs`

    # tune2fs -L logical-volume1 /dev/mapper/vg1-lv1

### For XFS

**Commands:**
+ xfs_admin - change parameters of an XFS filesystem

Changing the label to 'lg-vm2'

    # xfs_admin -L lg-vm2 /dev/mapper/vg1-lv2
    writing all SBs
    new label = "lg-vm2"

    # blkid /dev/mapper/vg1-lv2
    /dev/mapper/vg1-lv2: LABEL="lg-vm2" UUID="145c1d64-dc00-4568-9d6a-550cd60d035d" BLOCK_SIZE="512" TYPE="xfs"

---
[⬅️ Back](4-Configure-local-storage.md)
