8.d Configure superuser access
===

**Commands:**
+ visudo (8)           - edit the sudoers file
+ sudoers (5)          - default sudo security policy plugin

**Files:**
+ `/etc/sudo.conf` - Specifies security policies and plugin (not needed for the exam)
+ `/etc/sudoers` - Main configuration file for sudo (list of who can run what)
+ `/etc/sudoers.d/` - Drop-in files. Allows additional configuration files to be added, separately, on top of the main configuration file (`/etc/sudoers`)

**⚠️ WARNING:** *Always make sure to edit the sudoers files (`/etc/sudoers` or `/etc/sudoers.d/*`) with `visudo` as it checks for potential mistakes*

Example Usages
---

Allow root to run any commands anywhere

    root    ALL=(ALL)       ALL

Allows people in group wheel to run all commands

    %wheel  ALL=(ALL)       ALL

**📝 NOTE:** _Adding a user to the 'wheel' group (`usermod -a -G wheel user`) will effectively give him superuser access_

Allows people in group wheel to run all commands without a password

    %wheel        ALL=(ALL)       NOPASSWD: ALL

Read drop-in files from `/etc/sudoers.d` (the # here does not mean a comment)

    #includedir /etc/sudoers.d

The drop-in file below give the user 'victor' sudo access without the need of a password

    # cat /etc/sudoers.d/00_victor  
    victor    ALL=(ALL)       NOPASSWD: ALL

Give user1 sudo access to run `fdisk -l` and `reboot`

    user1    ALL=(ALL) /sbin/fdisk -l, /sbin/reboot, /bin/passwd, !/bin/passwd root

Give user1 sudo access to run `passwd` except `passwd root` (allows to change the password for all other users, except root)

    user1    ALL=(ALL) /bin/passwd, !/bin/passwd root

Give user sudo access to a list of commands via a command alias

    Cmnd_Alias STORAGE = /sbin/fdisk, /sbin/sfdisk, /sbin/parted, /sbin/partprobe, /bin/mount, /bin/umount
    user1 ALL = STORAGE

Allow user1 to run all commands as user2

    user1 ALL=(user2)    ALL

```
$ whoami
user1
$ sudo -i -u user2
$ whoami
user2
```

Show sudo access

```
$ sudo -l
Matching Defaults entries for user1 on rhel8-lab:
    !visiblepw, always_set_home, match_group_by_gid, always_query_group_plugin, env_reset,
    env_keep="COLORS DISPLAY HOSTNAME HISTSIZE KDEDIR LS_COLORS", env_keep+="MAIL PS1 PS2 QTDIR USERNAME
    LANG LC_ADDRESS LC_CTYPE", env_keep+="LC_COLLATE LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES",
    env_keep+="LC_MONETARY LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE", env_keep+="LC_TIME LC_ALL LANGUAGE
    LINGUAS _XKB_CHARSET XAUTHORITY", secure_path=/sbin\:/bin\:/usr/sbin\:/usr/bin

User user1 may run the following commands on rhel8-lab:
    (user2) ALL
```

---
[⬅️ Back](8-Manage-users-and-groups.md)
