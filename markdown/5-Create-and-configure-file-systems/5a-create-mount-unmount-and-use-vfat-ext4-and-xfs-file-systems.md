# 5.a Create, mount, unmount, and use vfat, ext4, and xfs file systems

## Introduction

### Superblock  

A superblock is a record of the characteristics of a filesystem, including its size, the block size, the empty and the filled blocks and their respective counts, the size and location of the inode tables, the disk block map and usage information, and the size of the block groups.

### Fstab

The file fstab contains descriptive information about the filesystems the system can mount.

**See man pages for:**
+ fstab (5)            - static information about the filesystems
+ mount (8)         - mount a filesystem

      #
      # /etc/fstab
      # Created by anaconda on Wed May 16 20:44:20 2018
      #
      # Accessible filesystems, by reference, are maintained under '/dev/disk'
      # See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
      #
      UUID=5f1871e2-c19c-4f86-8d6c-04d5fda71a0a /                       xfs     defaults        0 0
                        |                       |                        |          |           | |- FS Check Order (0 to ...)
                        |                       |                        |          |           |- Dump Frequency (backups)
                        |                       |                        |          |-  Mount Options
                        |                       |                        |- FSType
                        |                       |- Mount Point
                        |- Device Identifier

## Working with vfat Filesystems

**Commands:**
+ mkfs.vfat (8)        - create an MS-DOS filesystem under Linux
+ fsck.vfat (8)        - check and repair MS-DOS filesystems

### Creating and Mounting a VFAT Filesystem

Creating a vfat filesystem

    # mkfs.vfat -n VFAT /dev/vg3/vfat
    mkfs.fat 4.1 (2017-01-24)

Get the label

    # blkid | grep vfat
    /dev/mapper/vg3-vfat: LABEL="VFAT" UUID="E873-3EE5" BLOCK_SIZE="512" TYPE="vfat"

Add it to fstab

    UUID=E873-3EE5    /mnt/vfat    vfat    defaults    0 2

Mount it

    # mount –a

Confirm

    # df /mnt/vfat
    Filesystem           1K-blocks  Used Available Use% Mounted on
    /dev/mapper/vg3-vfat   3135488     4   3135484   1% /mnt/vfat

### Checking the VFAT filesystem

For VFAT, this can be done with the filesystem mounted.

    # fsck.vfat /dev/mapper/vg3-vfat
    fsck.fat 4.1 (2017-01-24)
    /dev/mapper/vg3-vfat: 1 files, 1/783872 clusters

## Working with XFS Flesystems

**Commands:**
+ mkfs.xfs (8)         - construct an XFS filesystem
+ xfs_repair (8)       - repair an XFS filesystem
+ xfs_info (8)         - display XFS filesystem geometry information
+ xfs_admin (8)        - change parameters of an XFS filesystem

### Creating and Mounting an XFS Filesystem

Create the filesystem

    # mkfs.xfs -L xfs /dev/mapper/vg2-xfs
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

Get the UUID

    # lsblk -f | grep xfs
    ├─sda1        xfs               375fcafb-dbf0-4e72-a1de-9bba811fc6d4   /boot
      ├─rhel-root xfs               cc50014d-ed1f-4785-a5f2-304ec6da9002   /
    └─vg2-xfs     xfs         xfs   cfc57e17-8108-4215-9de1-1ad8ffcf2326    
    └─vg2-xfs     xfs         xfs   cfc57e17-8108-4215-9de1-1ad8ffcf2326

Add it to fstab

    UUID=cfc57e17-8108-4215-9de1-1ad8ffcf2326 /mnt/xfs      xfs     defaults 0 2

Mount it

    # mount -a

Check it

    # df /mnt/xfs
    Filesystem          1K-blocks  Used Available Use% Mounted on
    /dev/mapper/vg2-xfs   6273024 76796   6196228   2% /mnt/xfs

### Checking the XFS filesystem

**📝 NOTE:** *The filesystem needs to be unmounted.*

Unmount if first

    # umount /mnt/xfs

Run `xfs_repair`

    # xfs_repair /dev/mapper/vg2-xfs
    Phase 1 - find and verify superblock...
    Phase 2 - using internal log
            - zero log...
            - scan filesystem freespace and inode maps...
            - found root inode chunk
    Phase 3 - for each AG...
            - scan and clear agi unlinked lists...
            - process known inodes and perform inode discovery...
            - agno = 0
            - agno = 1
            - agno = 2
            - agno = 3
            - process newly discovered inodes...
    Phase 4 - check for duplicate blocks...
            - setting up duplicate extent list...
            - check for inodes claiming duplicate blocks...
            - agno = 0
            - agno = 1
            - agno = 2
            - agno = 3
    Phase 5 - rebuild AG headers and trees...
            - reset superblock...
    Phase 6 - check inode connectivity...
            - resetting contents of realtime bitmap and summary inodes
            - traversing filesystem ...
            - traversal finished ...
            - moving disconnected inodes to lost+found ...
    Phase 7 - verify and correct link counts...
    done

### Settings XFS Flags

**Commands:**
+ xfs_admin (8)        - change parameters of an XFS filesystem

**Common options:**
+ `L` - Sets the FS label
+ `l` - Displays the FS label
+ `u`- Shows the current UUID
+ `U` - Sets the FS UUID
  + `nil` - set the filesystem UUID to the null UUID
  + `generate` - generate a  new  UUID for the filesystem
  + `restore` - restore the original UUID and remove the incompatible feature flag as needed
+ `c` - Enables/disables lazy counter

> With lazy counters enabled, the superblock is not modified or logged when changes are made to the free-space and inode counters. Information is stored in other parts of the file system to maintain the counter values. This provides significant performance improvements in some configurations. Enabling and disabling lazy counters is time-consuming on large file systems because the entire file system must be scanned.  

## Working with ext4 Filesystems

**Commands:**
+ mkfs.ext4 (8)        - create an ext2/ext3/ext4 filesystem
+ fsck.ext4 (8)          - check a Linux ext2/ext3/ext4 file system
+ tune2fs (8)            - adjust tunable filesystem parameters on ext2/ext3/ext4 filesystems
+ dumpe2fs (8)        - dump ext2/ext3/ext4 filesystem information

### Creating and Mounting an EXT4 Filesystem

Create the filesystem

    # mkfs.ext4 -L ext4 /dev/vg1/ext4
    mke2fs 1.45.6 (20-Mar-2020)
    Creating filesystem with 1570816 4k blocks and 393216 inodes
    Filesystem UUID: 3e636509-d28a-49fd-91ff-33b7f56f9757
    Superblock backups stored on blocks:  
        32768, 98304, 163840, 229376, 294912, 819200, 884736

    Allocating group tables: done                             
    Writing inode tables: done                             
    Creating journal (16384 blocks): done
    Writing superblocks and filesystem accounting information: done  

Get the label

    # lsblk -f | grep ext4
    └─vg1-ext4    ext4        ext4  3e636509-d28a-49fd-91ff-33b7f56f9757    
    └─vg1-ext4    ext4        ext4  3e636509-d28a-49fd-91ff-33b7f56f9757  

Add it to fstab

    UUID=3e636509-d28a-49fd-91ff-33b7f56f9757    /mnt/ext4    ext4     defaults    0 2

Mount it

    # mount –a

Check it

    # df /dev/mapper/vg1-ext4
    Filesystem           1K-blocks  Used Available Use% Mounted on
    /dev/mapper/vg1-ext4   6118976 24536   5763896   1% /mnt/ext4

### Checking the EXT4 filesystem

**📝 NOTE:** *The file system needs to be unmounted*

Unmount it

    # umount /mnt/ext4

Run the check

    # fsck.ext4 /dev/mapper/vg1-ext4
    e2fsck 1.45.6 (20-Mar-2020)
    ext4: clean, 11/393216 files, 47206/1570816 blocks

### Settings ext4 Flags

`tune2fs` allows the system administrator to adjust  various  tunable  filesystem  parameters  on Linux ext2, ext3, or ext4 filesystems.
+ `L` - Sets the FS label
+ `l` - Displays the FS label
+ `U` - Sets the FS UUID
  + `clear` - clear the filesystem UUID
  + `random` - generate a new randomly-generated UUID
  + `time` - generate a new time-based UUID

---

**📌 EXAM TIP:** *Use `man fs` for an overview of the filesystems shown here*

---
[⬅️ Back](5-Create-and-configure-file-systems.md)
