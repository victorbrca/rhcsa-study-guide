8.c Create, delete, and modify local groups and group memberships
===

Example Commands
---

### Getting Group Information

Get user's group

    $ id user2
    uid=1002(user2) gid=1002(user2) groups=1002(user2),100(users)

Or

    $ groups
    user2 users

Get user's primary group (you can also use the previous commands)

    # getent group user2
    user2:x:1002:

Get a list of all users and their groups

    # lslogins -G

Show all groups in the system

    # cat /etc/group

Or

    # getent [group]

Show users belonging to a group

    # lslogins -g [group]  

Or with groupmems (needs to be root to use '-g')

    # groupmems -l -g [group]

### Creating Groups

Create group with specified GID

    # groupadd -g [id] [name]

Create a system group

    # groupadd -r [name]


### Managing Groups

Change the group name

    # groupmod -n [new name] [group]

Change the group ID

    # groupmod -g [ID] [group]

**📝 NOTE:** *Changing a group id will result in group users not having access to existing files*

Add user to group (overwrites primary group)

    # usermod -g [group] [user]

Add user to secondary group (overwrites all secondary groups)

    # usermod -G [group]

Add user to secondary group (append to secondary without overwriting)

    # usermod -aG [group]

Add user to multiple secondary groups (overwrites existing secondary groups)

    # usermod -G [group1,group2,group3]

Remove user from secondary group

    # gpasswd -d [user] [group]

### Removing Groups

Use `groupdel`

    # groupdel [group]

**📝 NOTE:** *You cannot remove groups that are assigned as primary groups for existing users*

---
[⬅️ Back](8-Manage-users-and-groups.md)
