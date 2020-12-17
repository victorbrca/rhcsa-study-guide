1.j List, set, and change standard ugo/rwx permissions
===

## Permission in Linux

Linux divides the file permissions into 3 groups with read, write and execute denoted by r,w, and x respectively.

Each file and directory has three user based permission groups:

- **user** – The Owner permissions apply only the owner of the file or directory, they will not impact the actions of other users.
- **group** – The Group permissions apply only to the group that has been assigned to the file or directory, they will not effect the actions of other users.
- **others** – The All Users permissions apply to all other users on the system, this is the permission group that you want to watch the most.

And each user group has 3 permissions:

- **Read** - This permission give you the authority to open and read a file. Read permission on a directory gives you the ability to lists its content.
- **Write** - The write permission gives you the authority to modify the contents of a file. The write permission on a directory gives you the authority to add, remove and rename files stored in the directory as well as the directory itself.
- **Execute** - In Windows, an executable program usually has an extension ".exe" and which you can easily run. In Unix/Linux, you cannot run a program unless the execute permission is set. If the execute permission is not set, you might still be able to see/modify the program code (provided read & write permissions are set), but not run it. Fon a directory the execute permission allows you to `cd` into the directory.

      r = read permission
      w = write permission
      x = execute permission
      - = no permission

On the example below we observe the following permission:

    # ls -l my_file.txt
    -rwxr-xr-x. 1 root root 0 Dec  9 15:34 my_file.txt

- **User root** - Can read, write and execute
- **Group root** - Can read and execute
- **Others** - Can read and execute

Commands
---

### chmod

Takes octal (0000) and symbolic version (ugo,-+=,rws)

### chown

Changing ownership:

    chown root.cloud_user [file]

Which is the same as:

    chown root:cloud_user [file]

umask
---

**📝 NOTES:**
+ Default permission is '666' for files and '777' for directories
+ umask value is subtracted from the default permission (666)
+ umask is not persistent. To make it persistent, add it to `/etc/profile` (login shells), `/etc/bashrc` (non-login shells) or `~/.bash_profile`
+ First digit of umask (if specifying with 4 digits) is for special permission (sticky bit, etc.)

Example of umask configuration in `/etc/profile` so values are different for system accounts vs real users:

    # By default, we want umask to get set. This sets it for login shell
    # Current threshold for system reserved uid/gids is 200
    # You could check uidgid reservation validity in
    # /usr/share/doc/setup-*/uidgid file
    if [ $UID -gt 199 ] && [ "`/usr/bin/id -gn`" = "`/usr/bin/id -un`" ]; then
        umask 002
    else
        umask 022
    fi

Special Permissions
---

**📝 NOTE:** *It looks like special permission (except SGID) may not be part of the v8 exam*

### SUID - Set-user Identification

When a command or script with SUID bit set is run, its effective UID becomes that of the owner of the file, rather than of the user who is running it.

    rws-----

Note that SUID does not work on scripts that start with shebang ('#!').

    # chmod u+s [file]
    -rwsr--r--. 1 root root 0 Mar 16 21:48 test

    # chmod 4744 [file]
    -rwsr--r--. 1 root root 0 Mar 16 21:48 test

**📝 NOTE:** *A capital 'S' (`-rwSr--r--`) indicates that the execute bit is not set*

### SGID - Set-group identification

SGID permission is similar to the SUID permission, only difference is – when the script or command with SGID on is run, it runs as if it were a member of the same group in which the file is a member.

    rwxr-sr--

#### Setting SGID

    # chmod g+s [file]
    -rwxr-sr--. 1 root root 0 Mar 16 21:48 test

    # chmod 2754 [file]
    -rwxr-sr--. 1 root root 0 Mar 16 21:48 test

**📝 NOTE:** *A capital 'S' (`-rwxr-Sr--`) indicates that the execute bit is not set*


### Sticky bit  

Anyone can write, but only owner can delete the files (like '/tmp').

    drwxrwxrwt

Sticky bit is usually set on directories. Setting the sticky bit on a folder does nothing (on Linux).

#### Setting sticky bit

    # chmod o+t [file]
    drwxrwrwt. 1 root root 0 Mar 16 21:48 testdir

    # chmod 1777 [file]
    drwxrwxrwt. 1 root root 0 Mar 16 21:48 testdir

**📝 NOTES:**
+ A capital 'T' indicates that the execute bit is not set
+ You should give write permission to make sure that the target users can write to the folder

Cheat Table
---

| Mode       | Octal | Symbolic |
| ---------- | ----- | -------- |
| SUID       | 4755  | u+s      |
| SGID       | 2775  | g+s      |
| Sticky Bit | 1777  | o+t      |


Additional Special Permissions
---

A '.' can represent special permissions (SELinux related).

    -rw-rw-rw-.  

A '+' indicates ACLs are applied.

    -rw-rw-rw-+


---
[⬅️ Back](1-Understand-and-use-essential-tools.md)
