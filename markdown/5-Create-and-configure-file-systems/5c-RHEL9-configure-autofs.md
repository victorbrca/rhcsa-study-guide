# 5.c Configure autofs
Although network devices can be mounted at boot using `/etc/fstab`, mounts configured this way consume additional resources because they are always mounted and therefore don't scale well. `autofs` works with the kernel to mount drives only when they are accessed, improving performance.

*This topic includes:*
- Installing autofs
- Configuring autofs
- Debugging autofs

*Commands:*
- autofs(5)
- auto.master(5)

## Getting started
Begin by installing the `autofs` package, as this will install the main config files, daemon, and man pages.

```
dnf install -y autofs
```

The configuration files for autofs are stored in the `/etc/` folder. There is a fair amount of files, but understanding all of them is not necessary. `/etc/auto.master.d` holds the core configurations used.

```
ls /etc/auto*
/etc/autofs.conf  /etc/autofs_ldap_auth.conf  /etc/auto.master  /etc/auto.misc  /etc/auto.net  /etc/auto.smb

/etc/auto.master.d:
<your config>.autofs
```

**📌 EXAM TIP:** *All the files in `/etc/auto.master.d` need to end in .autofs. If you can't remember this there is an explanation and examples near the bottom of the `auto.master` man page.*

## Autofs syntax
Autofs is configured using two files. For the purposes of RHCSA, both the files are one-liners. The first file points to the actual configuration file.

<!-- - The first variable points to the desired mount directory. -->
<!-- - The second variable points to the config file loaded when the directory in the first variable is accessed. -->

```
# cat /etc/auto.master.d/opt.autofs
/opt    /etc/autofs.opt
```

The second file is a bit more complex. It defines the server mount point and client mount point as well as mount options. The following configuration maps all directories in the server's etc (using `&`) to `/opt/*`.

```
# cat /etc/autofs.opt
*   -rw example.org:/etc/&
```

## Debugging autofs
The above configuration maps all files and directories in the `/etc/` directory of the host to the `/opt/` directory on the client. 

Each time the files are changed, autofs will need to be reloaded:

```
systemctl restart autofs
systemctl status autofs
```

**📝 NOTE:** *Because the files are mounted when accessed, displaying the files can be confusing. It may seem like nothing has happened but the mounts are there.*

```
[root@client]:/opt# ls -a
. ..

[root@client]:/opt# cd /opt/lvm

[root@client]:/opt/lvm# ls
lvm.conf lvmlocal.conf profile

[root@client]:/opt/lvm# cd ../ && ls -a
. .. lvm
```

**📝 NOTE:** *Autofs depends on connecting to a network server. Configuring one isn't in the spec, but it's not far out of scope. You can either choose to understand autofs well from just notes, use autofs mounts locally, or apply your knowledge by building an NFS server.*






