8.a Create, delete, and modify local user accounts
===

**Important Files:**
+ `/etc/skell`                       - Skeleton folder
+ `/etc/passwd`                  - Password file
+ `/etc/group`                     - Group file
+ `/etc/default/useradd`   - Default variables for useradd
+ `/etc/login.defs`              - The /etc/login.defs file defines the site-specific configuration for the shadow password suite. This file is required. Absence of this file will not prevent system operation, but will probably result in undesirable operation.

*Note that the shadow file, no one has permission:*

    $ ll /etc/shadow
    ----------. 1 root root 887 Jan 29  2019 /etc/shadow

**Commands:**
+ id (1)                       - print real and effective user and group IDs
+ getent (1)               - get entries from Name Service Switch libraries
+ useradd (8)            - create a new user or update default new user information
+ usermod (8)          - modify a user account
+ whoami (1)            - print effective userid
+ logname (1)          - print user's login name
+ lslogins (1)         - display information about known users in the system


Commands
---

### id - Print real and effective user and group IDs

    $ id
    uid=1000(victor) gid=1000(victor) groups=1000(victor),10(wheel) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023

**📝 NOTE:** *When running for your own user, the highlighted text is shown*


### getent - get entries from Name Service Switch libraries

This can be used to show information on local and directory services.

Shows /etc/passwd entry for user 'victor'

    # getent passwd victor
    victor:x:1000:1000:victor:/home/victor:/bin/bash

Shows /etc/shadow entry for user 'user1'

    # getent shadow user1
    user1:$6$i5IgGA0XFP/f3Hv3$./zCBLGuL24fc8njwvsGJlsgrB.QJzf1srrHMUpDzxgYyqwlepHrKUhbu.k9iwOMxLsdRuu.oNiRItGZiKNZE.:18592:0:99999:7:::

Shows all users in the system

    # getent passwd

Shows all groups in the system

    # getent group

### useradd - Create a new user or update default new user information

**Useful command options:**
+ `c` - Comment
+ `d` - Set home directory
+ `g` - Set the GID
+ `G` - Secondary groups
+ `k` - Skeleton directory
+ `p` - The encrypted password, as returned by crypt
+ `r` - Create a system account
+ `s` - Default login shell
+ `u` - Set the UID

Add a user named 'user1' with uid '1001' and the existing 'users' group

    # useradd -u 1001 -g 100 user1

    # id user1
    uid=1001(user1) gid=100(users) groups=100(users)

### usermod - Modify a user account

**Useful command options:**
+ `c` - Modify user's comment
+ `d`- change the user's home directory
+ `G` - chage the user's secondary group
+ `L` - Locks the account
+ `U` - Unlocks the account

**📝 NOTE:** *Locking the account adds a '!' at the beginning of the hash in `/etc/shadow`*

---
[⬅️ Back](8-Manage-users-and-groups.md)
