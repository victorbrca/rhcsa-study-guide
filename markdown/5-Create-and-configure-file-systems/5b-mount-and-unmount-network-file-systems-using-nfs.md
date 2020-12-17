# 5.b Mount and unmount network file systems using NFS

## Definition

Network File System (NFS) is a distributed file system protocol originally developed by Sun Microsystems (Sun) in 1984,[1] allowing a user on a client computer to access files over a computer network much like local storage is accessed. NFS, like many other protocols, builds on the Open Network Computing Remote Procedure Call (ONC RPC) system.  

**Commands and Man Pages:**
+ nfs (5)                  - fstab format and options for the nfs file systems
+ exports (5)          - NFS server export table
+ exportfs (8)         - maintain table of exported NFS file systems
+ showmount (8)  - show mount information for an NFS server
+ mount.nfs (8)        - mount a Network File System
+ auto.master (5)      - Master Map for automounter consulted by autofs
+ autofs (5)           - Format of the automounter maps
+ autofs (8)           - Service control for the automounter
+ autofs.conf (5)      - autofs configuration

**📝 NOTES**
+ Make sure both client and server has `nfs-utils` installed
+ Avoid disabling the `no_root_squash` option (it's enabled by default and changes ownership of files from 'root' to 'nobody')

## Simple NFS Setup (fstab)

### On the Server

Install NFS on the server

    # dnf install -y nfs-utils

Add firewall rule for nfs, rpc-bind and mountd and reload

    # firewall-cmd --permanent --add-service=nfs --add-service=rpc-bind --add-service=mountd
    success

    # firewall-cmd --reload
    success

Create directory and setup permission

    # mkdir /nfs/fstab
    # chmod 777 /nfs/fstab

Edit `/etc/exports`

    /nfs/fstab *(rw)

**📌 EXAM TIP:** *If you can't remember the format for `/etc/exports` take a look at the examples at the end of `man exports`.*

Setup SELinux for NFS

    # setsebool -P nfs_export_all_rw on

Enable and start the service (this also starts 'rpcbind')

    # systemctl enable --now nfs-server

Run `exportfs -a` to update the NFS table and then run `showmount -e` to list the mounts

    # exportfs -a

    # showmount -e localhost
    Export list for localhost:
    /nfs/fstab *

**Additional Info on 'exportfs'**

> The  exportfs command maintains the current table of exports for the NFS server.  The master export table is kept in a file named /var/lib/nfs/etab.  This file  is  read  by  rpc.mountd when a client sends an NFS MOUNT request.
>
> Normally  the master export table is initialized with the contents of /etc/exports and files under /etc/exports.d by invoking exportfs -a.  However, a system administrator can choose to add  or delete exports without modifying /etc/exports or files under /etc/exports.d by using the exportfs command.

### On the Client

Install `nfs-utils` if needed

    # dnf install -y nfs-utils

Enable and `rpcbind`

    # systemctl enable --now rpcbind

Check that you can see the mount from the server with 'showmount -e [server_IP]'

    # showmount -e [server_IP]

Create the mount point

    # mkdir -p /mnt/nfs/fstab

Mount the share

    # mount -t nfs [server_IP]:[server_path] [mount_path]

#### Making the mount permanent

Unmount the share

Add it to `/etc/fstab`

    # NFS Mount
    10.0.2.15:/nfs/fstab            /mnt/nfs/fstab        nfs     defaults        0 2

Muount it

    # mount -a

Check that it was mounted

    # df -h | grep nfs
    10.0.2.15:/nfs/fstab    29G  2.5G   27G   9% /mnt/nfs/fstab

#### Additional Info

**Options:**
+ `no_root_squash` - Allows root users on client computers to have root access on the server. Mount requests for root are not be mounted to the anonymous user. This option is needed for diskless clients.
+ `root_squash` - Map  requests  from uid/gid 0 to the anonymous uid/gid. Note that this does not apply to any other uids or gids that might be equally sensitive, such as user bin or group staff.

## Configuring AutoFS

The configuration for autofs is done on the client. We will be configuring both indirect and direct maps, as well as the use of variables (like when setting autofs for the `$HOME` of users).

The main advantage of indirect map is that changes made to the map file are loaded automatically, while changes made to a direct map require a service reload.

**📝 NOTE:** *For the server configuration on all 3 examples, it's assumed that firewall services have been enabled and that the SELinux NFS RW boolean is also enabled.*

### Indirect

With indirect map we edit the master map file (`/etc/auto.master`) and specify a root mount point and a map file. In the map file we can specify one of more mount with options.

#### On the server

Let's create a new share on the server so we don't have to change our previous work

    # mkdir /nfs/auto_indirect

    # chmod 777 /nfs/auto_indirect

    # echo -e "/nfs/auto_indirect\t\t*(rw)" >> /etc/exports

    # systemctl reload nfs-server.service

    # exportfs -a

#### Client

Install autofs

    # yum install autofs

Create the base mount point

    # mkdir /mnt/nfs/autofs

Modify `/etc/auto.master`

    /mnt/nfs/autofs   /etc/auto.nfs --timeout 60
      |                    |              |- optional
      |                    |- Map file location  
      |- Root mount point on client (should exist)

**📌 EXAM TIP:** *If you can't remember the format look at the example at the end of `man auto.master`*

Next, create the indirect map file by adding the following line to `/etc/auto.nfs`

    auto_indirect       -fstype=nfs,rw            10.0.2.15:/nfs/auto_indirect
        |                     |                         |          |- Share
        |                     |                         |- Server
        |                     |- Additional parameters
        |- Subfolder of mount point (should not exist)

**📌 EXAM TIP:** *If you can't remember the format look at the example at the end of `man 5 autofs`*

Enable and start the autofs service

    # systemctl enable --now autofs.service

Check if you can browse to the subfolder `/mnt/nfs/autofs/auto_indirect`. The folder only gets created when you browse to it. Once the connection is stale the share is unmounted and the folder is removed.

### Direct

With direct we still specify a map file in the master map file, but the full mount point is specified in the map file.

Direct maps are specified with a `/-` in the master file.

#### On the server

Let's create a new share on the server

    # mkdir /nfs/auto_direct

    # chmod 777 /nfs/auto_direct

    # echo -e "/nfs/auto_direct\t\t*(rw)" >> /etc/exports

    # systemctl reload nfs-server.service

    # exportfs -a

#### Client

We already have the base folder

    # ls -l /mnt/nfs/autofs

Modify `/etc/auto.master` and set the key to `/-` (tells auto.master that it's a direct file), and add the map file location (`/etc/auto.direct`)

    /-                      /etc/auto.direct

In the auto.direct file we add the full local path, options and remote host and remote share

    /mnt/nfs/autofs/auto_direct     -fstype=nfs,rw     10.0.2.15:/nfs/auto_direct

Restart the autofs service and try to browse to the folder

    # systemctl restart autofs.service

    # cd /mnt/nfs/autofs/auto_direct

We can check all 3 mounts (from all 3 exercises)

    # df | grep nfs
    10.0.2.15:/nfs/fstab          30320256 2583936  27736320   9% /mnt/nfs/fstab
    10.0.2.15:/nfs/auto_direct    30320256 2583936  27736320   9% /mnt/nfs/autofs/auto_direct
    10.0.2.15:/nfs/auto_indirect  30320256 2583936  27736320   9% /mnt/nfs/autofs/auto_indirect

### Mounting Home

Another good example of auto mount is configuring remote folders for users. Here's how we can do that.

#### On the server

Let's create a new user on the server and specify the UID

    # useradd -u 1001 user1

Now let's create a new share

    # mkdir /nfs/home

Change the permission of the folder

    # chmod 777 /nfs/home

Copy the user's home folder (we cold also have specified the home folder when creating the new user with `useradd -u 1001 -d /nfs/home/user1 user1`)

    # cp -a /home/user1 /nfs/home/.

Add our share to `/etc/exports`

    # echo -e "/nfs/home\t\t*(rw)" >> /etc/exports

Reload the service

    # systemctl reload nfs-server.service

Update the nfs table

    # exportfs -a


#### Client

Create the mount

    # mkdir /mnt/nfs/home

Add the user with the same UID (we don't need to create a home folder)

    # useradd -M -u 1001 user1

Now we need to modify the user so his home is at `/mnt/nfs/home`. We can do that by changing `/etc/passwd`

    user1:x:1001:1001::/mnt/nfs/home/user1:/bin/bash
                      |change this section|

Add the config tothe master map file using `/etc/auto.home` as the map file

    /mnt/nfs/home           /etc/auto.home

Create the map file. Note that we are using a wild card (`*`) for the mount path, and a variable (`&`) for the remote path. The variable points to the user asking for the mount

    *       -fstype=nfs,rw          10.0.2.15:/nfs/home/&

Restart the autofs service

    # systemctl restart autofs.service

Try to login as the user and see if it works

    # su - user1

    $ pwd
    /mnt/nfs/home/user1

---
[⬅️ Back](5-Create-and-configure-file-systems.md)
