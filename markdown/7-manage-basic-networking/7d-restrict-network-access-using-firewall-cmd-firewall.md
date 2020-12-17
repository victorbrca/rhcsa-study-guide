# 7.d Restrict network access using firewall-cmd/firewall

**📝 NOTE:** _For firewall configuration with 'firewall-cmd', see "[9-Manage security => Configure firewall settings using firewall-cmd/firewalld](../9-Manage-security/9a-configure-firewall-settings-using-firewall-config-firewall-cmd-or-iptables.md)"_

**Reference:**
- firewalld (1)        - Dynamic Firewall Manager
- firewall-cmd (1)     - firewalld command line client

## Creating a new Zone and Adding Services and Interface

Create a new zone

    # firewall-cmd --permanent --new-zone=server
    success

Add the http service

    # firewall-cmd --permanent --zone=server --add-service=http
    success

Add an interface to the zone

    # firewall-cmd --change-interface=enp0s8 --zone=server --permanent
    The interface is under control of NetworkManager, setting zone to 'server'.
    success

Add another service to the zone

    # firewall-cmd --add-service=ssh --zone=server --permanent
    success

Reload the configuration

    # firewall-cmd --reload
    success

Check that the zone was added

    # firewall-cmd --get-zones
    block dmz drop external home internal libvirt nm-shared public server trusted work

Check the zone configuration

    # firewall-cmd --list-all --zone=server
    server (active)
      target: default
      icmp-block-inversion: no
      interfaces: enp0s8
      sources:  
      services: http ssh
      ports:  
      protocols:  
      masquerade: no
      forward-ports:  
      source-ports:  
      icmp-blocks:  
      rich rules:  

## Add a new Port

Add the port

    # firewall-cmd --add-port=8888/tcp --zone=server --permanent
    success

Confirm that the new rule was added

    # firewall-cmd --zone=server --list-ports --permanent  
    8888/tcp

Reload the configuration

    # firewall-cmd --reload
    success

Close a Port

    # firewall-cmd --remove-port=[port/protocol] {--permanent}

Reload the configuration

    # firewall-cmd --reload
    success

---
[⬅️ Back](7-manage-basic-networking.md)
