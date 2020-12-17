# 7.b Configure hostname resolution

## Hostname

**Commands:**
- hostnamectl (1)      - Control the system hostname
- hostname (1)         - show or set the system's host name

Viewing the hostname with 'hostname' - hostname {-f|-s}

    # hostname
    localhost.localdomain

Viewing the hostname with 'hostnamectl'

    # hostnamectl
       Static hostname: localhost.localdomain
             Icon name: computer-vm
               Chassis: vm
            Machine ID: b31bb9e7fc544e65beae56247bdd423f
               Boot ID: 876d1a6703d0469384df37a4855fa1f0
        Virtualization: oracle
      Operating System: Red Hat Enterprise Linux 8.3 (Ootpa)
           CPE OS Name: cpe:/o:redhat:enterprise_linux:8.3:GA
                Kernel: Linux 4.18.0-240.1.1.el8_3.x86_64
          Architecture: x86-64

### Change the Hostname  

#### hostname

When used with an argument, the command 'hostname' temporarily sets the hostname until the next reboot

    # hostname rhel8-lab

    # hostname
    rhel8-lab

#### /etc/hostname

You can edit '/etc/hostname' to change the hostname permanently.  

#### hostnamectl

Sets the hostname permanently.  

    # hostname rhel8-lab

    # hostname
    rhel8-lab

## DNS

**📌 EXAM TIP:** *For the test, make sure that 'bind-utils' is installed so you have the 'host' resolution utility.*

### Changing the DNS entry

Update it with 'nmcli'

    # nmcli con mod [connection name] ipv4.dns [DNS IP]

Restart the connection or the network

    # nmcli connection down [conn name] && nmcli connection up [conn name]

Or

    # systemctl restart NetworkManager

Optionally, if you need to add a secondary DNS server, add a '+' before 'ipv4.dns

    # nmcli con mod [connection name] +ipv4.dns [secondary DNS IP]

**📝 Notes**
- You can confirm the DNS changes in '/etc/resolv.conf' after restarting the connection or the network
- Using a '+' will add a second entry (max of 3)
- Using a '-' will remove the entry

### Overwriting DNS Resolutions

The '/etc/hosts' file is a simple text file that associates IP addresses with hostnames, one line per IP address. You can use '/etc/hosts' to overwrite DNS resolution (for programs that use the GNY C library).  

    # cat /etc/hosts
    127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
    ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

    127.0.0.1    myhost    myhost.com

**Reference:**
- hosts (5)            - static table lookup for hostnames

### DNS Resolutions

**Commands:**
- nsswitch.conf (5)    - Name Service Switch configuration file
- nslookup (1)         - query Internet name servers interactively
- host (1)             - DNS lookup utility
- dig (1)              - DNS lookup utility

#### Name Service Switch configuration file

The Name Service Switch (NSS) configuration file `/etc/nsswitch.conf` is used by the GNU C Library and certain other applications to determine the sources from which to obtain name-service information in a range of categories, and in what order.

By default, '/etc/hosts' (files) is given priority over DNS queries:

    # grep hosts /etc/nsswitch.conf
    #     hosts: files dns
    #     hosts: files dns  # from user file
    # Valid databases are: aliases, ethers, group, gshadow, hosts,
    hosts:      files dns myhostname

Both 'nslookup' and 'host' commands do not use C library, thus are not affected by nsswitch.conf (or `/etc/hosts`).

#### nslookup

You can use 'nslookup' for name resolution

    # nslookup google.ca
    Server:        10.13.15.1
    Address:    10.13.15.1#53
    Non-authoritative answer:
    Name:    google.ca
    Address: 172.217.165.3
    Name:    google.ca
    Address: 2607:f8b0:400b:802::2003

#### host

You can also use 'host' for name resolution

    # host google.ca
    google.ca has address 172.217.165.3
    google.ca has IPv6 address 2607:f8b0:400b:802::2003
    google.ca mail is handled by 40 alt3.aspmx.l.google.com.
    google.ca mail is handled by 10 aspmx.l.google.com.
    google.ca mail is handled by 30 alt2.aspmx.l.google.com.
    google.ca mail is handled by 50 alt4.aspmx.l.google.com.
    google.ca mail is handled by 20 alt1.aspmx.l.google.com.

#### dig

dig is an advanced DNS tool and is here mainly for reference  

    # dig google.ca +noall +answer
    google.ca.        210    IN    A    172.217.165.3

---
[⬅️ Back](7-manage-basic-networking.md)
