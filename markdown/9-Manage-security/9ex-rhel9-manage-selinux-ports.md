# 9.e Manage SELinux port labels
One way in which SELinux secures the systen is by monitoring ports. Specifically, it monitors the name of the processes which are running on each port and determines their SELinux Port Type. If the specified port is not in the list (nonstandard), that process is blocked.

*This topic includes:*
- Configuring SELinux port labels
- Short tcp, udp explanation
- Port label example

*Commands:*
- semanage-port(8)
- systemctl(1)

## View SELinux Port Labels
SELinux comes pre-configured with an large number of port-application mappings. You can interact with these mappings using the `semanage port` commands. To list them add the `-l` or `--list` argument.

```
semanage port -l | head
SELinux Port Type              Proto    Port Number

afs3_callback_port_t           tcp      7001
afs3_callback_port_t           udp      7001
afs_bos_port_t                 udp      7007
afs_fs_port_t                  tcp      2040
afs_fs_port_t                  udp      7000, 7005
afs_ka_port_t                  udp      7004
afs_pt_port_t                  tcp      7002
afs_pt_port_t                  udp      7002
```

**📌 EXAM TIP:** _To navigate through the list of ports on-the-fly, you can pipe the output into `less` and search for a port type or port number using `/`._

All port mappings use either the TCP or UDP protocol.

## TCP, UDP
Both protocols are essential in the modern world. People are generally more exposed to TCP applications. Here are some protocols based on each one:

| | TCP | UDP |
---|:---:|:---:
|Protocols|TCP is used by HTTP, HTTPs, FTP, SMTP and Telnet.| UDP is used by DNS, DHCP, TFTP, SNMP, RIP, and VoIP. |

Briefly, the key difference between these protocols is that TCP requires a handshake. For a bit more info on this topic [here is a more comprehensive table](https://www.geeksforgeeks.org/differences-between-tcp-and-udp/).

## Adding, Removing Port Labels

As a template, manage SELinux port labels as so:

- `semanage port -<a|d> -t <port_type> -p <tcp|udp> <port>`
  - `-a` : add a label
  - `-d` : delete a label
  - `-t` : port type
  - `-p` : protocol


## Managing SELinux ports example
In this example, a nonstandard port mapping will be added to allow a `httpd` service to run on it. 

1. Assume a working `httpd` server:
```
[root@server ~]$ dnf install -y httpd
[root@server ~]$ systemctl enable --now httpd
[root@server ~]$ curl localhost:80
# <h1>Hello World</h1>
```

2. Now, the server will be configured so that instead of listening on port 80, it listens on port 8012:

```
[root@server ~]$ vim /etc/httpd/conf/httpd.conf # holds port config. not needed in exam. for demonstration.
    # ...
    Listen 8012
    # ...
```

3. Restart httpd, an error occurs:

```
[root@server ~]$ systemctl restart httpd
# Job for httpd.service failed because the control process exited with error code.
# See "systemctl status httpd.service" and "journalctl -xeu httpd.service" for details.

[root@server ~]$ systemctl status httpd
× httpd.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)
     Active: failed (Result: exit-code) since Mon 2023-02-13 11:55:52 GMT; 6s ago
       Docs: man:httpd.service(8) ...

Feb 13 11:55:52 server systemd[1]: Starting The Apache HTTP Server...
Feb 13 11:55:52 server httpd[2174]: (13)Permission denied: AH00072: make_sock: could not bind to address [::]:8012
Feb 13 11:55:52 server httpd[2174]: (13)Permission denied: AH00072: make_sock: could not bind to address 0.0.0.0:8012
Feb 13 11:55:52 server httpd[2174]: no listening sockets available, shutting down
Feb 13 11:55:52 server systemd[1]: Failed to start The Apache HTTP Server.
```

4. Add the nonstandard port mapping to SELinux. 
```
semanage port -a -t http_port_t -p tcp 8012
```

5. Restart httpd
```
[root@server ~]$ systemctl restart httpd

[root@server ~]$ systemctl status httpd
● httpd.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)
     Active: active (running) since Mon 2023-02-13 11:58:59 GMT; 1h 27min ago
       Docs: man:httpd.service(8)

Feb 13 11:58:59 server systemd[1]: Starting The Apache HTTP Server...
Feb 13 11:58:59 server httpd[2255]: Server configured, listening on: port 8012
Feb 13 11:58:59 server systemd[1]: Started The Apache HTTP Server.
```

6. The service now listens on the desired port
```
[root@server ~]$ curl localhost:8012
# <h1>Hello World</h1>
[root@server ~]$ curl localhost:80
# curl: (7) Failed to connect to localhost port 80 after 0 ms: Connection refused
```

---
[⬅️ Back](9-manage-security.md)