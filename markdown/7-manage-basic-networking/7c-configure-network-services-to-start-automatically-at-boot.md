# 7.c Configure network services to start automatically at boot

## Enable a Service to Start on Boot

As we have seen before, use 'systemctl enable'

    # systemctl enable httpd
    Created symlink /etc/systemd/system/multi-user.target.wants/httpd.service → /usr/lib/systemd/system/httpd.service.

Confirmation that it's enabled

    # systemctl status httpd
    ● httpd.service - The Apache HTTP Server
       Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
       Active: inactive (dead)
         Docs: man:httpd.service(8)

## Enable a Network Connection to Start on Boot

With 'nmcli'

    # nmcli connection modify [conn name] autoconnect yes

---
[⬅️ Back](7-manage-basic-networking.md)
