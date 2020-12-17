1.d Access remote systems using SSH
===

## The SSH protocol

The SSH protocol uses encryption to secure the connection between a client and a server. All user authentication, commands, output, and file transfers are encrypted to protect against attacks in the network.

![](1d-Access-remote-systems-using-SSH/1d-Access-remote-systems-using-SSH-c3524.png)

The new protocol replaced several legacy tools and protocols, including telnet, ftp, FTP/S, rlogin, rsh, and rcp.

**Old Technologies that the SSH stack replaces**
+ Telnet -> SSH
+ RCP -> SCP
+ FTP -> SFTP

## Usage

SSH is typically used to log into a remote machine and execute commands, but it also can transfer files using the associated SSH file transfer (SFTP) or secure copy (SCP) protocols. SSH uses the client-server model.

### Basic Syntax

To connect to a remote system using SSH, we’ll use the ssh command. The most basic form of the command is:

    ssh remote_host

The remote_host in this example is the IP address or domain name that you are trying to connect to.

This command assumes that your username on the remote system is the same as your username on your local system.

If your username is different on the remote system, you can specify it by using this syntax:

    ssh remote_username@remote_host

Once you have connected to the server, you may be asked to verify your identity by providing a password.

**Important command flags/options**
+ `-l` - login
+ `-i` - specify a private key to use
+ `-F` - specify config file
+ `-X` - enable X forwarding

### Configuration Files

**Client Config files**
+ `~/.ssh/config`
+ `/etc/ssh/ssh_config`

**Server config file**
+ `/etc/ssh/sshd_config`

## Other Important SSH Commands

We will review these later.

+ `ssh-keygen`
+ `ssh-copy-id` *(note that this is a script and not a binary)*

## How to enable commonly used config

The listed configuration is done at the server: `/etc/ssh/sshd_config`.

+ X11 forwarding
  + `X11Forwarding yes`
+ sftp subsytem
  + `Subsystem  sftp    /usr/lib/ssh/sftp-server`
+ Root login
  + `PermitRootLogin yes`
+ Password authentication
  + `PasswordAuthentication yes`

---
[⬅️ Back](1-Understand-and-use-essential-tools.md)
