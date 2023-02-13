# 9.c Manage default file permissions
*This topic includes:*
- System base file permissions
- Change default file permissions
- Inspect file permissions

*commands*
- umask
- ls
- stat

## System file permission defaults.
By default, files in Linux have octal permissions of `0666` and folders have permissions of `0777`. 

These defaults are quite permissive and some permissions are often subtracted for better security.

**📝 NOTE:** *Folders are granted execute permissions by default because without this, users cannot `cd` into them. Execute permissions on files must be granted manually.*

Permissions can be automatically subtracted by configuring a `umask`. Most Linux systems, including RHEL, come pre-configured with a `umask` which can be viewed from the terminal.

```
umask
# 0002
```

The `umask` subtracts from the base permissions as shown in this diagram:

![Default permissions for a directory](9c-rhel9-manage-default-file-permissions/default_umask_table.png)

**📝 NOTE:** *The default mask for a standard user is `0002`; for the root user it is `0022`.*

## Change default file permissions
To changing the default file permissions is as simple as changing the `umask`. This can be done non-persistently by writing the octal value after the command:

```
umask 0077
umask
# 0077
```

**⚠️ WARNING:** _Changing the umask as above does not persist on reboot, or even on opening a new shell._

To always change the default umask for a user, you can add the command into their .bashrc:

```
echo 'umask 0077' >> /home/user/.bashrc
```

To change system-wide umask default, edit `/etc/login.defs`

```
vim /etc/login.defs
##
# ERASECHAR       0177
# KILLCHAR        025
# UMASK           022 -> 025
## 
:wq

/bin/bash

umask 
0025
```

**📝 NOTE:** *`umask` can only reduce default permissions, not increase them. Because of this, [special permissions](https://www.redhat.com/sysadmin/suid-sgid-sticky-bit) are not really affected by `umask` since they have a value of 0 by default.*

## Inspect file permissions
To confirm whether files made with a umask have the desired permissions, `ls -l` can show symbolic permissions and octal permissions can be viewed using stat.

```
umask 0077
touch file
stat file

# File: file
# Size: 0         	Blocks: 0          IO Block: 4096   regular empty file
# Device: 803h/2051d	Inode: 28968428    Links: 1
# Access: (0600/-rw-------)  Uid: ( 1000/      user)   Gid: ( 1000/      user)
```

[⬅️ Back](9-manage-security.md)