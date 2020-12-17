# 9.g Use boolean settings to modify system SELinux settings

## SELinux Booleans

Booleans are on/off settings for functions in SELinux. There are hundreds of settings that can turn SELinux capabilities on or off, and many are already predefined.  

### Listing Booleans

Use 'getsebool' to get current booleans and their values.

Get status on a specific boolean

    # getsebool httpd_enable_cgi
    httpd_enable_cgi --> on

List all booleans  

    # getsebool -a | head -n 4
    abrt_anon_write --> off
    abrt_handle_event --> off
    abrt_upload_watch_anon_write --> on
    antivirus_can_scan_system --> off

_You can also use 'semanage'_

    # semanage boolean -l | head -n 4
    SELinux boolean                State  Default Description
    abrt_anon_write                (off  ,  off)  Allow ABRT to modify public files used for public file transfer services.
    abrt_handle_event              (off  ,  off)  Determine whether ABRT can run in the abrt_handle_event_t domain to handle ABRT event scripts.

_Or 'sestatus -b'_

    # sestatus -b | tail -n 4
    zarafa_setrlimit                            off
    zebra_write_config                          off
    zoneminder_anon_write                       off
    zoneminder_run_sudo                         off

**Examples**

List all enabled booleans  

    # getsebool -a | grep ' on'

Getting info on SQL booleans  

    # getsebool -a | grep -i sql
    mysql_connect_any --> off
    mysql_connect_http --> off
    postgresql_can_rsync --> off
    postgresql_selinux_transmit_client_label --> off
    postgresql_selinux_unconfined_dbadm --> on
    postgresql_selinux_users_ddl --> on
    selinuxuser_mysql_connect_enabled --> off
    selinuxuser_postgresql_connect_enabled --> off

Getting additional information on boolean (not part of RHCSA v8 and requires the `setools-console` package)

    # sesearch -b httpd_execmem -A
    allow httpd_suexec_t httpd_suexec_t:process { execmem execstack }; [ httpd_execmem ]:True
    allow httpd_sys_script_t httpd_sys_script_t:process { execmem execstack }; [ httpd_execmem ]:True
    allow httpd_t httpd_t:process { execmem execstack }; [ httpd_execmem ]:True

### Changing Boolean Values

Booleans can be set for runtime or persistent (survives reboots)

#### Runtime

Configures a policy for runtime

    # setsebool httpd_enable_cgi on

In some systems you can also use `togglesebool`

    # togglesebool httpd_enable_cgi

**💡 TIP:** _switching booleans on runtime only is fast and helps you to debug problems_

#### Persistent

To make the change persistent, use the `-P` option with `setsebool`

    # setsebool -P httpd_enable_cgi on  

---
[⬅️ Back](9-manage-security.md)
