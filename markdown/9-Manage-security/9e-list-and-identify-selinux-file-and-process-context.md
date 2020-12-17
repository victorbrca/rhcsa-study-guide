# 9.e List and identify SELinux file and process context

**Commands:**
- ls (1)               - list directory contents
- secon (1)            - See an SELinux context, from a file, program or user input.

## SELinux Contexts

All files, directories, devices have a security context/label associated with them.   These context  are stored in the extended attributes of the file system.  

### List All Contexts

    # semanage fcontext -l

### Viewing Context of Files

You can use the 'ls' command to view context of files.  

Viewing context of files

    # ls -Z
        system_u:object_r:admin_home_t:s0 anaconda-ks.cfg
        system_u:object_r:admin_home_t:s0 initial-setup-ks.cfg
    unconfined_u:object_r:admin_home_t:s0 install.file
    unconfined_u:object_r:admin_home_t:s0 install.file.rpm
    unconfined_u:object_r:admin_home_t:s0 my_repo
    unconfined_u:object_r:admin_home_t:s0 test_file.txt

Viewing context for a file with long listing

    # ls -lZ /etc/ssh/ssh_config.d/05-redhat.conf  
    -rw-r--r--. 1 root root system_u:object_r:etc_t:s0 831 Feb  4 16:01 05-redhat.conf

Viewing the context of a file with 'secon'

    # secon -f /etc/ssh/ssh_config.d/05-redhat.conf  
    user: system_u
    role: object_r
    type: etc_t
    sensitivity: s0
    clearance: s0
    mls-range: s0

### Viewing Context of Processes

Viewing context for a process (`ps auxZ`, `ps -efZ` or `ps -efM`)

    # ps -efZ | grep httpd
    system_u:system_r:httpd_t:s0    root     12462     1  0 17:42 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
    system_u:system_r:httpd_t:s0    apache   12463 12462  0 17:42 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
    system_u:system_r:httpd_t:s0    apache   12464 12462  0 17:42 ?        00:00:01 /usr/sbin/httpd -DFOREGROUND
    system_u:system_r:httpd_t:s0    apache   12465 12462  0 17:42 ?        00:00:01 /usr/sbin/httpd -DFOREGROUND
    system_u:system_r:httpd_t:s0    apache   12466 12462  0 17:42 ?        00:00:01 /usr/sbin/httpd -DFOREGROUND
    system_u:system_r:httpd_t:s0    apache   12803 12462  0 18:45 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
    unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 root 12883 11680  0 18:47 pts/0 00:00:00 grep --color=auto httpd

Displaying process context with 'secon'

    # secon -p 1495
    user: system_u
    role: system_r
    type: httpd_t
    sensitivity: s0
    clearance: s0
    mls-range: s0
---
[⬅️ Back](9-manage-security.md)
