# 9.c Configure key-based authentication for SSH

## Overview of SSH

**Packages**
- openssh-server
- openssh-clients

**Service**
- sshd.service

**Server Configuration Files**
- System - /etc/ssh/sshd_config
- User - ~/.ssh/authorized_keys

**Client Configuration Files**
- System - /etc/ssh/ssh_config
- User - ~/.ssh/config

**Commands:**
- sshd_config (5)      - OpenSSH SSH daemon configuration file
- ssh_config (5)        - OpenSSH SSH client configuration files
- ssh (1)                     - OpenSSH SSH client (remote login program)
- ssh-keygen (1)       - authentication key generation, management and conversion
- ssh-copy-id (1)       - use locally available keys to authorize logins on a remote machine

## Server Configuration

Commonly used configuration (good to know). They reside in '/etc/ssh/sshd_config'.  

Enable 'root' login

    PermitRootLogin yes

Enable password authentication (keyless login)

    PasswordAuthentication yes

Change the port for SSH

    Port 22

Change the listen IP for SSH

    ListenAddress 0.0.0.0

## Keyless Login

With keyless login you specify the remote user and the remote server to login to. SSH will prompt for the remote user password (if password login is allowed in '/etc/ssh/sshd_config').  

    # ssh user@server

## Key-based Login

a. First you need to create the keys on the client with 'ssh-keygen'

This command will:
- Create the ~/.ssh folder with the proper permissions (0700)
- Create the public key (used on the server) with the proper permissions (0600)
- Create the private key (used on the client machine)  

**📝 NOTES:**  
- You can give a passphrase (similar to a password) to the key upon creation. You will need to supply the passphrase when trying to access the server
- Multiple key types can be used (RSA, DSA, etc.)

      # ssh-keygen
      Generating public/private rsa key pair.
      Enter file in which to save the key (/root/.ssh/id_rsa):  
      Created directory '/root/.ssh'.
      Enter passphrase (empty for no passphrase):  
      Enter same passphrase again:  
      Your identification has been saved in /root/.ssh/id_rsa.
      Your public key has been saved in /root/.ssh/id_rsa.pub.
      The key fingerprint is:
      SHA256:UKiiZrSSEpTHK4OTavsyrXjLe2DXUk6nF4IXYuZgPzY root@rhel8-lab
      The key's randomart image is:
      +---[RSA 3072]----+
      |  o    ..        |
      | ooo+ o.         |
      |oo.*.+..         |
      |=+..E =.o        |
      |o=+o O +S.       |
      |*=o o + .        |
      |*.oo . .         |
      |.=...            |
      |.oO=             |
      +----[SHA256]-----+

b. After you can use the `ssh-copy-id` script to copy the public key to the server

The script will:
+ Attempt to login with the key (to avoid copies)
+ Prompt you for the remote user password
+ Create `~/.ssh` on the remote server with the right permission (0700)
+ Create `~/.ssh/authorized_keys` (if needed) on the remote server with the right permission (0600)
+ Copy the public key to `~/.ssh/authorized_keys` on the remote server

      # ssh-copy-id root@server
      /bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/id_rsa.pub"
      The authenticity of host 'server (::1)' can't be established.
      ECDSA key fingerprint is SHA256:+Smq+fuyAF6UYeB0C7SxZSVgUg/s/gOFziZlh7dhA+o.
      Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
      /bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
      /bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
      root@server's password:  
      Number of key(s) added: 1

      Now try logging into the machine, with:   "ssh 'root@server'"
      and check to make sure that only the key(s) you wanted were added.
---
[⬅️ Back](9-manage-security.md)
