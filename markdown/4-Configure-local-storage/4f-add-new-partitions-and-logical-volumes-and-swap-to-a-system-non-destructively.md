# 4.f Add new partitions and logical volumes, and swap to a system non-destructively

### Adding a New Swap Logical Volume

**Commands:**
+ mkswap (8)           - set up a Linux swap area
+ swapon (2)           - start/stop swapping to file/device

Create a physical volume

    # pvcreate /dev/sdd  
      Physical volume "/dev/sdd" successfully created.

Create the volume group

    # vgcreate swap2 /dev/sdd
      Volume group "swap2" successfully created

Create the logical volume

    # lvcreate -l 100%FREE -n swap2 swap2
      Logical volume "swap2" created.

Get the lv path  

    # lvdisplay swap2 | grep 'LV Path'
      LV Path                /dev/swap2/swap2

Create the swap area

    # mkswap -L swap2 /dev/swap2/swap2
    Setting up swapspace version 1, size = 3 GiB (3217027072 bytes)
    LABEL=swap2, UUID=02dac232-4bb8-4f18-8a29-c5c77756aaa0

You can list the swap devices for comparison

    # swapon -s
    Filename                Type        Size    Used    Priority
    /dev/dm-1                                  partition    2158588    36800    -2

Enable the device for paging

    # swapon /dev/swap2/swap2

Compare the list of swap device again

    # swapon -s
    Filename                Type        Size    Used    Priority
    /dev/dm-1                                  partition    2158588    36800    -2
    /dev/dm-4                                  partition    3141628    0    -3

Add it to fstab

    /dev/swap2/swap2   none                    swap    defaults        0 0

**📌 EXAM TIP**

Be prepared to:
- Add a standard partition to a device and mount it ([4.a List, create, delete partitions on MBR and GPT disks](4a-list-create-delete-partitions-on-mbr-and-gpt-disks.md))
- Resize a standard partition ([5.c Extend existing logical volumes](..//5-Create-and-configure-file-systems/5-Create-and-configure-file-systems.md))
- Add a new device to an existing volume group and create a new logical volume partition

---
[⬅️ Back](4-Configure-local-storage.md)
