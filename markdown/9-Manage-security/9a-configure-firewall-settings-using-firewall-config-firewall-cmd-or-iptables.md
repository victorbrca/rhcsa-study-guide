# 9.a Configure firewall settings using firewall-config, firewall-cmd, or iptables

## IPv4 Forwarding

I'm not sure why this is a topic that is covered on many online and study sources, but it appears that enabling IP forwarding is part of the RHCSA exam, even thou it's not listed in the exam objectives (it may be left over from RHCSA v7 exam).  

Enabling IP forwarding is something that you would on a router, or potentially on a VPN server. It allows incoming traffic directed to a different IP to flow through the configured interface. With firewalld enabled, you would also need to configure rules for this to work.  

### Enabling IPv4 Forwarding

#### Runtime

    # sysctl -w net.ipv4.ip_forward=1
    net.ipv4.ip_forward = 1

#### Permanent

Add 'net.ipv4.ip_forward = 1' to '/etc/sysctl.conf

Load sysctl

    # sysctl -p
    net.ipv4.ip_forward = 1

**📌 EXAM TIP:** _If you can't remember the option, run `sysctl -a | grep ipv4 | grep forward` to get a list of options_

## Definitions

Firewall interaction in kernel is handled by netfilter

>"Netfilter is a framework provided by the Linux kernel that allows various networking-related operations to be implemented in the form of customized handlers."

Pre RHEL 7, `iptables` was the way to interact with netfilter.  

Today, `firewall-cmd` (`firewalld`) is the new way to interact with netfilter.  

## firewalld and firewall-cmd

Install the two packages if needed:
- firewalld
- firewall-config

Start and enable the service if needed.

### Firewalld Service

Listing service status via systemd

    # systemctl status firewalld.service
    ● firewalld.service - firewalld - dynamic firewall daemon
       Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; vendor preset: enabled)
       Active: active (running) since Wed 2020-11-25 14:45:39 EST; 3 days ago
         Docs: man:firewalld(1)
    Main PID: 1155 (firewalld)
        Tasks: 3 (limit: 12285)
       Memory: 11.3M
       CGroup: /system.slice/firewalld.service
               └─1155 /usr/libexec/platform-python -s /usr/sbin/firewalld --nofork --nopid

Listing service status via firewall-cmd

    # firewall-cmd --state
    running

### Runtime vs Persistent (Permanent)

The runtime configuration is the actual effective configuration and applied to the firewall in the kernel. At firewalld service start the permanent configuration becomes the runtime configuration. Changes in the runtime configuration are not automatically saved to the permanent configuration.

- **Runtime** - Change is automatic
- **Permanent** - Change stays with reboot

To remove current changes, or reload changes (overrides all runtime changes with config from permanent config)

    # firewall-cmd --reload

To save changes to permanent, without adding it to runtime (for port 80 TCP)

    # firewall-cmd --add-port=80/tcp --permanent

Add changes to runtime

    # firewall-cmd --add-port=80/tcp  

To save changes from runtime to permanent

    # firewall-cmd --runtime-to-permanent

Save changes to permanent and then to runtime

    # firewall-cmd --add-port=80/tcp --permanent

    # firewall-cmd --reload

### Firewall Zones

The firewalld daemon manages groups of rules using entities called “zones”. Zones are basically sets of rules dictating what traffic should be allowed depending on the level of trust you have in the networks your computer is connected to.

#### Commands

List zones

    # firewall-cmd --get-zones

Get default zone

    # firewall-cmd --get-default-zone

List everything added for or enabled in all zones

    # firewall-cmd --list-all-zones

List everything added for or enabled in zone

**📝 NOTES** *Most of the commands, without a zone name will list/change the default zone*

    # firewall-cmd --list-all {--zone=[zone]}

Add the http service (runtime)

    # firewall-cmd --add-service=http {--zone=[zone]}

Add the http service (permanent, not runtime)

    # firewall-cmd --add-service=http --permanent {--zone=[zone]}

### Services

List enabled services in a zone

    # firewall-cmd {--zone=[zone]} --list-services

List all predefined services

    # firewall-cmd --get-services {--zone=[zone]}

Get predefined ports of a service

    # firewall-cmd --permanent --service=ssh --get-ports
    22/tcp

## GUI Interface

You can also use a GUI (`firewall-config`) to configure the firewall. The UI is very intuitive.

It may not be installed by default, but you should be able to install it.  

    # dnf list firewall-config
    Updating Subscription Management repositories.
    Last metadata expiration check: 0:00:59 ago on Wed 16 Dec 2020 09:01:00 AM EST.
    Available Packages
    firewall-config.noarch                    0.8.2-2.el8                    rhel-8-for-x86_64-appstream-rpms

![](9a-configure-firewall-settings-using-firewall-config-firewall-cmd-or-iptables/9a-configure-firewall-settings-using-firewall-config-firewall-cmd-or-iptables-97b60.png)

### Using firewall-confid via SSH

RHEL 8 by default is using Wayland. While you should not have any issues in running `firewall-config` on the box, if for some reason you need to run it via SSH, follow the steps below.

If X11 does not work, check if either X11 or Wayland are installed `rpm -qa`. If they are not, you will need to install:
- xorg-x11-server-Xorg
- xorg-x11-xauth

Make sure that X11 forwarding (`/etc/ssh/sshd_config`) and/or X11 trusted forwarding (`/etc/ssh/ssh_config.d/05-redhat.conf`) are enabled.  

SSH with `-X` or `-Y` and you should be able to run `firewall-config`.



---

#### Additional Info

**See man pages for:**
- firewalld (1)            - Dynamic Firewall Manager
- firewall-cmd (1)     - firewalld command line client
- firewall-config (1)  - firewalld GUI configuration tool

---
[⬅️ Back](9-manage-security.md)
