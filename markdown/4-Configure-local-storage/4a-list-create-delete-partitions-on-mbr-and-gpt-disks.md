# 4.a List, create, delete partitions on MBR and GPT disks

**Commands to know for the exam:**
+ `df`
+ `lsblk`
+ `blkid`
+ `fdisk`
+ `gdisk`
+ `partprobe`

## MBR and GPT

MBR (Master Boot Record) and GPT (GUID Partition Table) are two different ways of storing the partitioning information on a drive. This information includes where partitions start and begin, so your operating system knows which sectors belong to each partition and which partition is bootable.

## MBR Partitions

A master boot record (MBR) is a special type of boot sector at the very beginning of partitioned computer mass storage devices like fixed disks or removable drives intended for use with IBM PC-compatible systems and beyond. The concept of MBRs was publicly introduced in 1983 with PC DOS 2.0.

MBR - Master Boot Record
+ Can only hold 4 primary partitions
+ Can only address up to 4TB of disk space

### Manipulating MBR Partitions

**Commands:**
- fdisk (8)            - manipulate disk partition table
- partprobe (8)        - inform the OS of partition table changes

List all disks and partitions

    # fdisk -l

List partitions on device

    # fdisk -l /dev/sda

#### Command Mode

Allows you to manipulate/create a disk and it's paritions.

Enter command mode

    # fdisk [disk]

**Options:**
- `m` - print help menu
- `a` - toggle a bootable flag
- `p` - print partition table
- `d` - delete partition
- `l` - list known partition types
- `n` - add a new partition
- `t` - change a partition type
- `w` - write table to disk and exit
- `q` - quit without saving changes

**📝 NOTE:** *You should be familiar using all this options to manage paritions*


## GPT Partitions

The GUID Partition Table (GPT) is a standard for the layout of partition tables of a physical computer storage device, such as a hard disk drive or solid-state drive, using universally unique identifiers, which are also known as globally unique identifiers (GUIDs).

Newer servers using UEFI can address GPT partitions, however older servers still using BIOS need additional software installed.

GPT partitions can be significantly larger than MBR partitions:
+ Can have nearly unlimited number of partitions
+ Partition size is limited to what the OS can address (RHEL has certified GPT partitions on XFS at 500 TiB and theoretical up to 16EiB)

### Manipulating GPT Partitions

**Commands:**
- gdisk (8)            - Interactive GUID partition table (GPT) manipulator

**📝 NOTE:** *Options are the same as on fdisk*

---
[⬅️ Back](4-Configure-local-storage.md)
