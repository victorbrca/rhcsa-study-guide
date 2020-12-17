# 9.h Diagnose and address routine SELinux policy violations

## Troubleshooting SELinux Issues

### Sealert (setroubleshoot)

`sealert` is the user interface component (either GUI or command line) to the setroubleshoot system. setroubleshoot is used to diagnose SELinux denials and attempts to provide  user  friendly explanations  for  a SELinux denial (e.g. AVC) and recommendations for how one might adjust the system to prevent the denial in the future.

Requires the packages:
- setroubleshoot-server
- setroubleshoot-plugins


**📌 EXAM TIP:** _Use `dnf provides sealert` if you can't remember the package name_

You can ask `sealert` to analyze errors in a log file

    # sealert -a /var/log/audit/audit.log  
    100% done
    found 0 alerts in /var/log/audit/audit.log

If problems are found, 'sealert' will provide with a possible solution

    --------------------------------------------------------------------------------                          
    SELinux is preventing /usr/bin/chcon from using the mac_admin capability.                                 
    *****  Plugin catchall (100. confidence) suggests   **************************                            
    If you believe that chcon should have the mac_admin capability by default.                                
    Then you should report this as a bug.                                                                     
    You can generate a local policy module to allow this access.                                             
    Do                                                                                                       
    allow this access for now by executing:                                                                  
    # ausearch -c 'chcon' --raw | audit2allow -M my-chcon                                                    
    # semodule -X 300 -i my-chcon.pp                                                                         
    Additional Information:                                                                                  
    Source Context                unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1                         
                                  023                                                                        
    Target Context                unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1                          
                                  023                                                                         
    Target Objects                Unknown [ capability2 ]                                                    
    Source                        chcon                                                                      
    Source Path                   /usr/bin/chcon                                                             
    Port                          <Unknown>                                                                   
    Host                          <Unknown>                                                                  
    Source RPM Packages           coreutils-8.30-8.el8.x86_64                                                 
    Target RPM Packages                                                                                       
    SELinux Policy RPM            selinux-policy-targeted-3.14.3-54.el8.noarch                               
    Local Policy RPM              selinux-policy-targeted-3.14.3-54.el8.noarch                               
    Selinux Enabled               True                                                                       
    Policy Type                   targeted                                                                   
    Enforcing Mode                Enforcing                                                                  
    Host Name                     rhel8-lab
    Platform                      Linux rhel8-lab 4.18.0-240.1.1.el8_3.x86_64 #1 SMP                         
                                  Fri Oct 16 13:36:46 EDT 2020 x86_64 x86_64
    Alert Count                   1                                                                          
    First Seen                    2020-12-01 13:47:36 EST
    Last Seen                     2020-12-01 13:47:36 EST                                                    
    Local ID                      c7afc8cf-fce0-44e6-9c78-cb72ee617c6b

### ausearch

`ausearch` is a tool that can query the audit daemon logs ('/var/log/audit/audit.log') for events based on different search criteria.   

Search all denials  

    # ausearch -m avc  

Search for denials for today  

    # ausearch -m avc -ts today  

Search for denials from the last 10 minutes  

    # ausearch -m avc -ts recent  

To search for SELinux denials for a particular service

    # ausearch -c httpd

Looking at the same error that we saw with 'sealert' for 'chcon'

    # ausearch -c chcon   
    ----
    time->Tue Dec  1 13:47:36 2020
    type=PROCTITLE msg=audit(1606848456.637:8732): proctitle=6368636F6E002D740074656D705F7400746573745F66696C652E747874
    type=SYSCALL msg=audit(1606848456.637:8732): arch=c000003e syscall=188 success=no exit=-22 a0=55ae4005e4f0 a1=7f52bd1fadde a2=55ae4005fbf0 a3=20 items=0 ppid=3146 pid=43817 auid=1000 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=pts0 ses=4 comm="chcon" exe="/usr/bin/chcon" subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 key=(null)
    type=SELINUX_ERR msg=audit(1606848456.637:8732): op=setxattr invalid_context="unconfined_u:object_r:temp_t:s0"
    type=AVC msg=audit(1606848456.637:8732): avc:  denied  { mac_admin } for  pid=43817 comm="chcon" capability=33  scontext=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 tcontext=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 tclass=capability2 permissive=0

### Systemd

You can also use `journalctl` to get information on SELinux issues (looking at the same message that we saw in `sealert` for `chcon`)

    Dec 01 13:47:42 rhel8-lab setroubleshoot[43819]: SELinux is preventing chcon from using the mac_admin capability. For complete SELinux messages run: sealert -l e30f49b8-93b4-403f-9fc2-f0cf4aa98732
    Dec 01 13:47:42 rhel8-lab setroubleshoot[43819]: SELinux is preventing chcon from using the mac_admin capability.
                                                     *****  Plugin catchall (100. confidence) suggests   **************************
                                                     If you believe that chcon should have the mac_admin capability by default.
                                                     Then you should report this as a bug.
                                                     You can generate a local policy module to allow this access.
                                                     Do
                                                     allow this access for now by executing:
                                                     # ausearch -c 'chcon' --raw | audit2allow -M my-chcon
                                                     # semodule -X 300 -i my-chcon.pp

### Getting More Information

We can use `audit2why` to get a description from a denied message.

Here we analyze the same `chcon` message that we saw before

    # ausearch -c chcon  | audit2why  
    type=AVC msg=audit(1606848456.637:8732): avc:  denied  { mac_admin } for  pid=43817 comm="chcon" capability=33  scontext=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 tcontext=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 tclass=capability2 permissive=0

        Was caused by:
            Missing type enforcement (TE) allow rule.

            You can use audit2allow to generate a loadable module to allow this access.

You can also list all the errors with

    # auditwhy -a

## Adding a New Module

Sometimes adding a new context might not be the best option, so you can create and install a new module instead.  

Run `ausearch` for a specific command name (for example, httpd). This should give you additional information on something that might be blocked by SELinux.  

    # ausearch -c 'httpd'

You can then pipe the output of that commad to `audit2allow` (which generates SELinux policy from logs)

    # ausearch -c 'httpd' --raw | audit2allow -M myhttpd

This generates 2 module files, 'myhttpd.pp' (binary) and 'myhttpd.te' (text)

To enable the module use

    # semodule -i myhttpd.pp

---
[⬅️ Back](9-manage-security.md)
