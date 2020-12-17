# 5.d Create and configure set-GID directories for collaboration

This includes setting SGID and adding users to the same group.  

**Commands:**
+ chmod (1)            - change file mode bits
+ usermod (8)          - modify a user account
+ useradd (8)          - create a new user or update default new user information


## SGID - Set-group identification

SGID permission is similar to the SUID permission, only difference is – when the script or command with SGID on is run, it runs as if it were a member of the same group in which the file is a member.

We can use SGID to allow for collaboration in a folder. When new files are created it will

    rwxr-sr--

Setting SGID

    # chmod g+s [file]
    -rwxr-sr--. 1 root root 0 Mar 16 21:48 test

    # chmod 2754 [file]
    -rwxr-sr--. 1 root root 0 Mar 16 21:48 test

**📝 NOTE:** *A capital 'S' (`-rwxr-Sr--`) indicates that the execute bit is not set*

## Setting a collaboration folder

Create the group

    # groupadd collab

Create two users

    # useradd -u 1001 user1
    # useradd -u 1002 user2

Add the two users to the group

    # gpasswd -a user1 collab

    # gpasswd -a user2 collab

Create the folder

    # mkdir /mnt/collab

Change the ownership

    # chown root.collab /mnt/collab

Set group write and SGID on the folder

    # chmod g+ws /mnt/collab

Login as the two users and create files in the folder. You should be able to have full access to all the files in the folder as any of the two users.

---
[⬅️ Back](5-Create-and-configure-file-systems.md)
