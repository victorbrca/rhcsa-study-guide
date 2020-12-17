# 3.j Securely transfer files between systems

**Commands:**
+ SCP - Same as SSH but for files (built into SSH). Same flags can be used
+ SFTP - OpenSSH comes with a built-in SFTP server. By default it's already enabled

## SCP

SCP or secure copy allows secure transferring of files between a local host and a remote host or between two remote hosts. It uses the same authentication and security as the Secure Shell (SSH) protocol from which it is based. SCP is loved for it’s simplicity, security and pre-installed availability.

### Examples

Copy file from local host to a remote host

    $ scp file.txt username@to_host:/remote/directory/

Copy file from a remote host to local host

    $ scp username@from_host:file.txt /local/directory/

Copy directory recursively from local host to a remote host

    $ scp -r /local/directory/ username@to_host:/remote/directory/

Copy directory recursively from a remote host to local host

    $ scp -r username@from_host:/remote/directory/  /local/directory/


**📌 TIP:** *When using `scp` with SSH keys setup between the local and remote systems, `[TAB]` will auto complete the path on the remote system*

## SFTP

The SSH File Transfer Protocol (SFTP), also known as the Secure File Transfer Protocol, enables secure file transfer capabilities between networked hosts. Unlike the Secure Copy Protocol (SCP), SFTP additionally provides remote file system management functionality, allowing applications to resume interrupted file transfers, list the contents of remote directories, and delete remote files.

To enable 'sftp' you might need to edit `/etc/ssh/sshd_config`

    # override default of no subsystems
    Subsystem   sftp    /usr/libexec/openssh/sftp-server


### Commands

Connect

    $ sftp user@host

Get help

    sftp> ?

---
[⬅️ Back](3-Operate-running-systems.md)
