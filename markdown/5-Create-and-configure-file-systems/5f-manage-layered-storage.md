# 5.f Manage layered storage

[RHEL > 8 > Managing storage devices > Chapter 19. Managing layered local storage with Stratis](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/managing_storage_devices/managing-layered-local-storage-with-stratis_managing-storage-devices)

## Stratis Definition

Stratis automates the management of local storage. On a system with just a single disk, Stratis can make it more convenient to logically separate /home from /usr, and enable snapshot with rollback on each separately. On larger configurations, Stratis can make it easier to create a multi-disk, multi-tiered storage pool, monitor the pool, and then manage the pool with less administrator effort.

Stratis is not a traditional filesystem like ext4, XFS, or FAT32. Stratis manages block devices and filesystems to support features akin to “volume-managing filesystems” (VMFs) like ZFS and Btrfs. Whereas a traditional filesystem acts to support directory and file operations on top of a single block device, VMFs can incorporate multiple block devices into a “pool”. Multiple independent filesystems can be created, each backed by the storage pool.

![](5f-manage-layered-storage/image.png)

### Supported block devices

Stratis pools have been tested to work on these types of block devices:  
+ LUKS  
+ LVM logical volumes  
+ MD RAID  
+ DM Multipath  
+ iSCSI  
+ HDDs and SSDs  
+ NVMe devices        

## Commands and Usage

### Getting Started

Install `stratid` and `stratis-cli`

    # dnf install -y stratis-cli stratisd

Enable and start the service

    # systemctl enable --now stratisd.service

### Pool

Create pool named 'pool1'

    # stratis pool create pool1 /dev/sdd /dev/sde

List pool

    # stratis pool list
    Name                  Total Physical   Properties
    pool1   6 GiB / 41.63 MiB / 5.96 GiB      ~Ca,~Cr

List devices in pool

    # stratis blockdev list  
    Pool Name   Device Node   Physical Size   Tier
    pool1       /dev/sdd              3 GiB   Data
    pool1       /dev/sde              3 GiB   Data

Add device to pool

    # stratis pool add-data pool1 /dev/sde

Delete the pool

    # stratis pool destroy pool1

### Filesystem

Create a filesystem labelled 'fs1' on 'pool1'

    # stratis fs create pool1 fs1

When we create a new filesystem, stratis formats it with XFS and we do not specify the size. The filesystem acts as a Thin LVM Volume that can dynamically grow to the Pool Size (we only use the space that we need). Each new filesystem will use close to 500MiB of storage space for the XFS log.

List the filesystem

    # stratis fs list
    Pool Name   Name   Used      Created             Device               UUID                             
    pool1       fs1    546 MiB   Nov 21 2020 21:06   /stratis/pool1/fs1   0b685a81af254ec38688c4d3bc3523f1

Delete the filesystem

    # stratis fs destroy pool1 fs1

### Snapshots

#### Creating Snapshots

Create a snapshot

    # stratis fs snapshot pool1 fs1 snapshot_$(date '+%Y%m%d_%H%M')

List snapshots

    # stratis fs list
    Pool Name   Name                     Used      Created             Device                                  UUID                             
    pool1       fs1                      546 MiB   Nov 21 2020 21:06   /stratis/pool1/fs1                      0b685a81af254ec38688c4d3bc3523f1
    pool1       snapshot_20201121_2117   546 MiB   Nov 21 2020 21:17   /stratis/pool1/snapshot_20201121_2117   bf903b2f29b54a579c5e0b4947a536f0

**📝 NOTE:** *You can mount the snapshot independently*

#### Restoring a snapshot

Optionally, back up the current state of the file system to be able to access it later

    # stratis filesystem snapshot pool1 fs1 fs1-backup

Unmount and remove the original file system:  

    # umount /stratis/pool1/fs1

    # stratis filesystem destroy pool1 fs1

Create a copy of the snapshot under the name of the original file system:  

    # stratis filesystem snapshot pool1 snapshot_20201121_2117 fs1

Mount the snapshot, which is now accessible with the same name as the original file system:  

    # mount /stratis/pool1/fs1 [mount-point]

Deleting a snapshot

    # stratis fs destroy snapshot pool1 snapshot_20201121_2117

## Overview of All Steps

Start by Installing Stratis

    # dnf install -y stratis-cli stratisd

Start and enable the service

    # systemctl enable --now stratisd.service

Create the pool  

    # stratis pool create pool1 /dev/sdd /dev/sde

Create the filesystem

    # stratis fs create pool1 fs1

Get the UUID and mount the filesystem (assuming the mount point exists)

    # blkid | grep stratis
    /dev/sde: UUID="aee3ff3fd5224f83a2241a70fd77d7ae" POOL_UUID="1225651149d040bdbbfdc74135f075e1" BLOCKDEV_SECTORS="6291456" BLOCKDEV_INITTIME="1606010576" TYPE="stratis"
    /dev/sdd: UUID="5927888d9f8e46babb3587e55657cbf4" POOL_UUID="1225651149d040bdbbfdc74135f075e1" BLOCKDEV_SECTORS="6291456" BLOCKDEV_INITTIME="1606010558" TYPE="stratis"
    /dev/mapper/stratis-1-1225651149d040bdbbfdc74135f075e1-thin-fs-f8cebd43c677467a847543d756685305: UUID="f8cebd43-c677-467a-8475-43d756685305" BLOCK_SIZE="512" TYPE="xfs"

Add it to fstab

    UUID=f8cebd43-c677-467a-8475-43d756685305       /mnt/stratis    xfs     defaults,x-systemd.requires=stratisd.service    0 2

> You can use `/stratis/<pool name>/<file system name>` but each time you rename a pool or file system you will need to update `/etc/fstab`, thus using file system UUID is recommended.

Regenerate mount units so that your system registers the new configuration (this is in the official documentation by Red Hat, however it should not be needed as we are not using systemd mount files)

    # systemctl daemon-reload

Try mounting the file system to verify that the configuration works   

    # mount -a

---

**References:**
+ https://stratis-storage.github.io/
+ https://www.theurbanpenguin.com/stratis-storage-management/


---
[⬅️ Back](5-Create-and-configure-file-systems.md)
