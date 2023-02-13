# 9.b Create and use file access control lists

## ACLS

**Commands:**
- acl (5)              - Access Control Lists
- getfacl (1)          - get file access control lists
- setfacl (1)          - set file access control lists

ACLs allows for a more granular permission structure on top of the existing file permission.  It is designed to assist with the default/basic UNIX file permissions. ACL allows you to give permissions for any user or group to any disk resource.
While a 'dot' indicates that it has an extended attribute (usually related to 'selinux')

    # ls -l /etc/profile
    -rw-r--r--. 1 root root 2078 Sep 10  2018 /etc/profile

A plus it indicates that an ACL is applied to it.

    # ll logins.txt
    -rwxr-xr--+ 1 root root 4.1K Nov 29 14:40 logins.txt

You can get the current ACL attribute of a file with 'getfacl':

    # getfacl /etc/profile
    getfacl: Removing leading '/' from absolute path names
    # file: etc/profile
    # owner: root
    # group: root
    user::rw-
    group::r--
    other::r--

Example 1

    # ll
    total 0
    drwxr-xr-x+ 2 smith    agents     6 Mar 10 19:41 agents
    drwxr-xr-x. 2 morpheus resistance 6 Mar 10 19:41 resistance

The folder 'agents' has:
- Basic permission: u:rwx, g:rx, o:rx
- ACL
  - Group resistance: rx

        # getfacl *
        # file: agents
        # owner: smith
        # group: agents
        user::rwx
        group::r-x
        group:resistance:r-x
        mask::r-x
        other::r-x

The folder 'resistance' has:
- Basic permission: u:rwx, g:rx, o:rx
- ACL: none

      # file: resistance
      # owner: morpheus
      # group: resistance
      user::rwx
      group::r-x
      other::r-x

### Masks

ACL masks show the current maximum permission. The ACL permission will always be dependent on standard permission.

    # ll logins.txt
    -rwxr-xr--+ 1 root root 4.1K Nov 29 14:40 logins.txt

The file 'logins.txt' has:
- Basic permission: u:rwx, g:rx, o:r
- ACL
  - Group users: rx
  - Group it-sup: w (modified by the mask essentially removing all permission)
  - Mask: rx

        # getfacl logins.txt
        # file: logins.txt
        # owner: root
        # group: root
        user::rwx
        group::r-x
        group:users:r--
        group:it-sup:-w-        #effective:---
        mask::r-x
        other::r--

### Enabling ACL

ACL should be enabled by default on newer versions of ext4 and xfs. If it's not, you can enable as a mount option:

    UUID=d207977a-8541-4cd3-b4e4-ea9b9ef13ca9    /mnt/ext4    ext4     defaults,acl    0 2

## Commands

### Adding permission

Add user permission ACL (`-m` to modify)

    # setfacl -m u:username:rwx [file]

Add group permission ACL

    # setfacl -m g:groupname:rwx [file]

You can also set the basic permission with setfacl

    # setfacl -m g::rw logins.txt

Is the same as

    # chomod g=rw logins.txt

When adding ACL permission, you can specify multiple rules

    # setfacl u::rw-,u:lisa:rw-,g::r--,g:toolies:rw-,m::r--,o::r-- [file]

Adding a mask

    # setfacl -m m:rw [file]

Set a default ACL for new files created. All new files in the directory will have the specified ACL

    # setfacl -d -m u:username:rwx [dir]

**📝 NOTE:** _Only directories can have default ACL_

### Removing ACL

Remove specific ACL

    # setfacl -x u:username:rwx [file]

Remove all ACLs

    # setfacl -b [file]

Use '-R' to run recursively

    # setfacl -R -m u:username:rwx [file]

Remove the default ACL

    # setfacl -k [dir]

### Other Options

Copy ACL from one file to another

    # getfacl [file1] | setfacl --set-file=- file2

Backup ACL of all files in a folder

    # getfacl -R [dir] > [backup file]

Restore ACL for all files in a folder from a backup file (you don't have to specify the folder. Just make sure you are in the parent folder of the target folder)

    # setfacl --restore=[backup file]
---
[⬅️ Back](9-manage-security.md)
