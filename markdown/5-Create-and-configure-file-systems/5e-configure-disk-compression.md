# 5e-configure-disk-compression

## Definition

Virtual Data Optimizer (VDO) provides inline data reduction for Linux in the form of deduplication, compression, and thin provisioning. When you set up a VDO volume, you specify a block device on which to construct your VDO volume and the amount of logical storage you plan to present.  

In the Red Hat Enterprise Linux 7.5 Beta, we introduced virtual data optimizer (VDO). VDO is a kernel module that can save disk space and reduce replication bandwidth. VDO sits on top of any block storage device and provides zero-block elimination, deduplication of redundant blocks, and data compression.  

VDO can be applied to a block device, and then normal disk operations can be applied to that device. LVM for example, can sit on top of VDO.

Physical disk -> VDO -> Volume group -> Logical volume -> filesystem

![](5e-configure-disk-compression/image.png)

## Requirements and Recommendations

### Memory

Each VDO volume has two distinct memory requirements:  

The VDO module

VDO requires 370 MB of RAM plus an additional 268 MB per each 1 TB of physical storage managed by the volume.  

The Universal Deduplication Service (UDS) index

UDS requires a minimum of 250 MB of DRAM, which is also the default amount that deduplication uses.  

The memory required for the UDS index is determined by the index type and the required size of the de-duplication window:       

![](5e-configure-disk-compression/image2png)

**📝 NOTE:** *Sparse is the recommended configuration.*

### Storage

#### Logical Size

Specifies the logical VDO volume size. The VDO Logical Size is how much storage we tell the OS that we have. Because of reduction and deduplication, this number will be bigger than the real physical size. This ratio will vary according to the type of data that is being stored (binary, video, audio, compressed data will have a very low ratio).  

##### Red Hat's Recommendation

_**For active VMs or container storage**_

Use logical size that is ten times the physical size of your block device. For example, if your block device is 1TB in size, use 10T here.  

_**For object storage**_

Use logical size that is three times the physical size of your block device. For example, if your block device is 1TB in size, use 3T here.            

#### Slab Size

Specifies the size of the increment by which a VDO is grown. All of the slabs for a given volume will be of the same size, which may be any power of 2 multiple of 128 MB up to 32 GB. At least one entire slab is reserved by VDO for metadata, and therefore cannot be used for storing user data.  

The default slab size is 2 GB in order to facilitate evaluating VDO on smaller test systems. A single VDO volume may have up to 8096 slabs. Therefore, in the default configuration with 2 GB slabs, the maximum allowed physical storage is 16 TB. When using 32 GB slabs, the maximum allowed physical storage is 256 TB.

![](5e-configure-disk-compression/image3.png)
_The table above is from RHEL 7 documentation_

#### Examples of VDO System Requirements by Physical Volume Size

The following tables provide approximate system requirements of VDO based on the size of the underlying physical volume. Each table lists requirements appropriate to the intended deployment, such as primary storage or backup storage.    

![](5e-configure-disk-compression/image4.png)

![](5e-configure-disk-compression/image5.png)

### Deduplication, Indexing and Compression

#### Deduplication and Index

VDO uses a high-performance deduplication index called UDS to detect duplicate blocks of data as they are being stored.

The UDS index provides the foundation of the VDO product. For each new piece of data, it quickly determines if that piece is identical to any previously stored piece of data. If the index finds match, the storage system can then internally reference the existing item to avoid storing the same information more than once.

Deduplication is enabled by default.  

To disable deduplication during VDO block creation (so only compression is used), use the `--deduplication=disabled` option (you will not be able to use the 'sparseIndex' option)

    # vdo create --name=[name] --device=/dev/[device] --vdoLogicalSize=[VDO logical size] --deduplication=disabled

To enable/disable deduplication on an existing block

    # vdo enableDeduplication --name=my_vdo

    # vdo disableDeduplication --name=my_vdo

#### Compression

In addition to block-level deduplication, VDO also provides inline block-level compression using the HIOPS Compression™ technology.  

VDO volume compression is on by default.

Compression operates on blocks that have not been identified as duplicates. When unique data is seen for the first time, it is compressed. Subsequent copies of data that have already been stored are deduplicated without requiring an additional compression step.

### Write Policy

VDO supports different write modes. These modes can be set with `--writePolicy=policy`. The write policy can be specified during the creation of a VDO, or when modifying an existing VDO volume with the `changeWritePolicy` subcommand.

- `sync` - Writes are acknowledged only after the data is guaranteed to persist.
- `async`  - Writes are acknowledged when the data has been cached for writing to the underlying storage. Data which has not been flushed is not guaranteed to persist in this mode, however this mode is ACID  compliant  (after  recovery  from  a  crash  any unflushed  write  is guaranteed either to have persisted all its data, or to have done nothing). Most databases and filesystems should use this mode.
- `async-unsafe` - Writes are handled like 'async' but there is no guarantee of the atomicity  async provides.  This mode should only be used for better performance when atomicity is not required.
- `auto` - VDO will check the storage device and determine whether it supports  flushes.  If it  does, VDO will run in async mode, otherwise it will run in sync mode. This is the default.

Setting the write policy when creating a VDO

    # vdo create --name=[name] --device=/dev/[device] --vdoLogicalSize=[VDO logical size] --writePolicy=sync

Changing the write policy on an existing VDO

    # vdo changeWritePolicy --writePolicy=sync --name=[name]

### Automatic Mounting

[RHEL > 8 > Deduplicating and compressing storage > Chapter 1. Deploying VDO > 1.9 Mounting a VDO volume](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/deduplicating_and_compressing_storage/deploying-vdo_deduplicating-and-compressing-storage#mounting-a-vdo-volume_deploying-vdo)

Like more other filesystems, there are two ways to mount filesystems (on top of VDO blocks) automatically during system startup: either via `/etc/fstab`, or adding a mount unit to systemd (on systemd-based systems).

For `/etc/fstab` mounting, in order to make sure the mount waits for the VDO service to start, use the mount option `x-systemd.requires=vdo.service`. For example, an `/etc/fstab` line involving VDO could be the following:

    /dev/mapper/vdo0 /vdo xfs defaults,discard,_netdev,x-systemd.device-timeout=0,x-systemd.requires=vdo.service 0 0

To add a mount it via systemd, modify the example in `/usr/share/doc/vdo/examples/systemd/VDO.mount.example` to match your mount point and configuration, and add it to `/etc/systemd/system` (and enabled/start it).

For the exam, see man pages for 'systemd.mount':

    _netdev
         Normally the file system type is used to determine if a mount is a "network mount", i.e. if
         it should only be started after the network is available. Using this option overrides this
         detection and specifies that the mount requires network

    x-systemd.device-timeout=
         Configure how long systemd should wait for a device to show up before
         giving up on an entry from /etc/fstab. Specify a time in seconds or
         explicitly append a unit such as "s", "min", "h", "ms".

    x-systemd.requires=
         Configures a Requires= and an After= dependency between the created mount
         unit and another systemd unit, such as a device or mount unit.

## Overview of Configuration

Install 'vdo' (and if not installed by default 'kmod-vdo')

    # yum install vdo kmod-vdo

Start/enable the service

    # systemctl enable --now vdo.service

Create the volume

    # vdo create --name=vdo1 --device=/dev/sdg --vdoLogicalSize=30G
    Creating VDO vdo1
          The VDO volume can address 6 GB in 3 data slabs, each 2 GB.
          It can grow to address at most 16 TB of physical storage in 8192 slabs.
          If a larger maximum size might be needed, use bigger slabs.
    Starting VDO vdo1
    Starting compression on VDO vdo1
    VDO instance 0 volume is ready at /dev/mapper/vdo1

**📝 NOTE:** *Using `--sparseIndex=disabled` will enable 'dense' indexing*

Optionally add LVM config,

    # pvcreate /dev/mapper/vdo1
      Physical volume "/dev/mapper/vdo1" successfully created.

    # vgcreate vdo1 /dev/mapper/vdo1
      Volume group "vdo1" successfully created

    # lvcreate -l +100%FREE -n vdo1 vdo1
      Logical volume "vdo1" created.

Create the file system (make sure to use the option to not discard blocks)

> Normally, when a filesystem is created, it runs a trim operation on the device. When using VDO, this is not ideal since the disk capacity is allocated on-demand. So we want to tell mkfs to not discard blocks during filesystem creation. For XFS, use the -K option, and for EXT4, use “-E nodiscard”.

**📝 NOTE:** *We specify `nodiscard` when creating the filesystem on top of VDO. However, for mounting, as shown on `/usr/share/doc/vdo/examples/systemd/VDO.mount.example`, we the `discard` option.*

    # mkfs.ext4 -E nodiscard [LV Path|VDO dev mapper]

    # mkfs.xfs -K [LV Path|VDO dev mapper]

If needed, update the system with the new device

    # udevadm settle

Mount the device

    # mount [LV Path|VDO dev mapper] /mount/point

To add it to `/etc/fstab`. As we mentioned before, you will need to add additional params so that systemd waits for VDO to start before mounting

    [LV Path|VDO dev mapper] /mount/point [fstype] defaults,discard,_netdev,x-systemd.device-timeout=0,x-systemd.requires=vdo.service 0 2

## Administration

Check for real physical space usage  

    # vdostats --human-readable

      Device              Size   Used   Available   Use%   Space Saving%
      /dev/mapper/my_vdo  1.8T  407.9G    1.4T       22%       21%

And you can view detailed information with 'vdo status'

    # vdo status

## Extending VDO Disks

**📝 NOTE:** *You cannot shrink the physical size of a VDO volume*

Check the VDO logical size (with 'df') and what VDO is actually configured with (`vdostats --human-readable`).

#### Grow Physical

Grow the VDO to its maximum size. This requires growing of the physical disk and due to the complexity it most likely will not show in the exam.

    # vdo growPhysical --name=[VDO name]

#### Grow Logical

If you have found that the compression or de-duplication of data was better than planned for you can increase the vdoLogicalSize of a device allowing VDO to show more space to the filesystem.

    # vdo growLogical --name=[VDO name] -vdoLogicalsize=[size]

Extend the file system (`xfs_growfs`, `resize2fs`)

---

### Additional Info

See `/usr/share/doc/vdo/` for additional information

**References:**
+ https://www.linuxsysadmins.com/setting-up-virtual-data-optimizer-on-centos/
+ https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/system_design_guide/deduplicating_and_compressing_storage#deploying-vdo_system-design-guide
+ https://www.theurbanpenguin.com/vdo-data-optimizer/

---
[⬅️ Back](5-Create-and-configure-file-systems.md)
