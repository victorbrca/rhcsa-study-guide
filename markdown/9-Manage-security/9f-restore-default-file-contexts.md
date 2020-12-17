# 9.f Restore default file contexts

## Changing SELinux Context

### Temporarily Changing Context

You can use `chcon` to temporarily change the context of files.  

**⚠️ WARNING:** _Changes with `chcon` will not survive `restorecon` or system relabel._

Changing the context of a file

    # chcon unconfined_u:object_r:tmp_t:s0 test_file.txt

Changing just the context type of a file

    # chcon -t [context_type] [file]

For example

    # ls -lZ /tmp/test
    -rw-r--r--. 1 root root unconfined_u:object_r:user_tmp_t:s0 403 Dec  1 13:48 /tmp/test

Change the context from 'user_tmp_t' to 'tmp_t'

    # chcon -t tmp_t /tmp/test  

Confirm the changes

    # ls -lZ /tmp/test
    -rw-r--r--. 1 root root unconfined_u:object_r:tmp_t:s0 403 Dec  1 13:48 /tmp/test

Changing a the context of a directory recursively

    # chcon -R [context] [dir]

### Persistently Changing Context

Make change persistent, even after relabeling (use absolute path)

    # semanage fcontext -a -t [context_type] [/absolut/path/to/file]  

    # restorecon [/absolut/path/to/file]  

Changes context of all files in '/root/my_web' (existing and future files) (use absolute path)

    # semanage fcontext -a -t httpd_sys_content_t '/root/my_web(/.*)?'

    # restorecon -R my_web

    # lz my_web/
    unconfined_u:object_r:httpd_sys_content_t:s0 httpd

️**⚠️ WARNING:** _`semanage` only changes SELinux database. After running `semanage` you will need to run `restorecon` to apply the configuration from the SELinux DB._  

## Restore Context

### Restore File Context

`restorecon` can also be run at any other time to correct inconsistent labels, to add support for newly installed  policy  or, by using the -n option, to passively check whether the file contexts are all set as specified by the active policy (default behavior).

If a file object does not have a context, restorecon will write the default context to the file object's  extended  attributes. If a file object has a context, restorecon will only modify the type portion of the security context.  The -F option will force a  replacement  of  the  entire context.

Relabeling will restore back context for files. You can restore the context for specific files and directories, or for the whole system (like when booting with 'rd.break' after reseting the root password).  

Restore context of a file

    # restorecon [file]

Restore the context of a directory recursively with verbose

    # restorecon -R -v [dir]

### Restore System Context / System Relabel and Restore

You can restore the SELinux context for the whole filesystem with 3 ways:

#### autorelabel

    # touch /.autorelabel

    # reboot

#### fixfiles

'fixfiles onboot' will setup the machine to relabel on the next reboot.

    # fixfiles onboot

    # reboot

#### On boot

Use the 'autorelabel' boot parameter to force a system relabel.  

- `autorelabel=1`
  - This parameter forces the system to relabel similarly to the previous commands

---
[⬅️ Back](9-manage-security.md)
