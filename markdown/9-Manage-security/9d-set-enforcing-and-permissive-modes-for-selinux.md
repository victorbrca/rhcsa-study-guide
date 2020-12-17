# 9.d Set enforcing and permissive modes for SELinux

**Commands:**
- selinux (8) - NSA Security-Enhanced Linux (SELinux)
- sestatus (8) - SELinux status tool
- setenforce (8) - modify the mode SELinux is running in
- getenforce (8) - get the current mode of SELinux
- selinuxenabled (8) - tool to be used within shell scripts to determine if selinux is enabled
- system-config-selinux (8) - SELinux Management tool (GUI)

## SELinux Definition

Security-Enhanced Linux is a Linux kernel security module that provides a mechanism for supporting access control security policies, including mandatory access controls. SELinux is a set of kernel modifications and user-space tools that have been added to various Linux distributions.

SELinux defines what process can have access to what files on a system. It does that by labeling every file, port, and socket with a context.  

### How does SELinux work?

SELinux defines access controls for the applications, processes, and files on a system. It uses security policies, which are a set of rules that tell SELinux what can or can’t be accessed, to enforce the access allowed by a policy.  

When an application or process, known as a subject, makes a request to access an object, like a file, SELinux checks with an access vector cache (AVC), where permissions are cached for subjects and objects.

If SELinux is unable to make a decision about access based on the cached permissions, it sends the request to the security server. The security server checks for the security context of the app or process and the file. Security context is applied from the SELinux policy database. Permission is then granted or denied.

If permission is denied, an "avc: denied" message will be available in /var/log/messages.

### Labeling

SELinux works as a labeling system, which means that all of the files, processes, and ports in a system have an SELinux label associated with them. Labels are a logical way of grouping things together. The kernel manages the labels during boot.

Labels are in the format 'user:role:type:level' (level is optional). User, role, and level are used in more advanced implementations of SELinux, like with MLS. Label type is the most important for targeted policy.  

### Modes

- **enforcing** - SELinux security policy is enforced.
- **permissive** - SELinux prints warnings instead of enforcing. Great for troubleshooting.  
- **disabled** - No SELinux policy is loaded

**⚠️ WARNING:** _Disabling SELinux is strongly discouraged. Best approach is to run it in 'permissive' mode and work to fix possible issues._  

### Policies

You can define which policy you will run by setting the SELINUXTYPE environment variable within `/etc/selinux/config`.   You  must  reboot  and possibly relabel if you change the policy type to have it take effect on the system.  The corresponding policy configuration for each such policy must be installed in the `/etc/selinux/{SELINUXTYPE}/` directories.

- **targeted** - Targeted processes are protected (default)
- **minimum** - Modification of targeted policy. Only selected processes are protected.  
- **mls** - Multi Level Security protection.

## Working With Modes

### Viewing the Current Mode

Use `getenforce`

    # getenforce  
    Enforcing

Or for more info, `sestatus`

    # sestatus
    SELinux status:                 enabled
    SELinuxfs mount:                /sys/fs/selinux
    SELinux root directory:         /etc/selinux
    Loaded policy name:             targeted
    Current mode:                   enforcing
    Mode from config file:          enforcing
    Policy MLS status:              enabled
    Policy deny_unknown status:     allowed
    Memory protection checking:     actual (secure)
    Max kernel policy version:      32

### Changing Modes

If you are enabling SELinux for the first time it's advisable to enable (preferably set to permissive) it in `/etc/selinux/config`, set the filesystem to auto relabel and then reboot.  

#### Via Command Line

Allows you to change between permissive and enforcing.

To enable enforcing  

    # setenforce = 1

To enable permissive

    # setenforce = 0

#### On Next Boot

Edit `/etc/selinux/config` and set `SELINUX=` to either 'enforcing', 'permissive' or 'disabled'

    SELINUX=enforcing

Reboot

    # reboot

#### Changing SELinux Modes at Boot Time

On boot, you can set several kernel parameters to change the way SELinux runs:

+ `enforcing=1`
   + Setting this parameter causes the machine to boot in enforcing mode.
+ `enforcing=0`
  + Setting this parameter causes the machine to boot in permissive mode, which is useful when troubleshooting issues.  
+ `selinux=0`
  + This parameter causes the kernel to not load any part of the SELinux infrastructure.  

**⚠️ WARNING:** _Using the selinux=0 parameter is not recommended. To debug your system, prefer using permissive mode._

---
[⬅️ Back](9-manage-security.md)
