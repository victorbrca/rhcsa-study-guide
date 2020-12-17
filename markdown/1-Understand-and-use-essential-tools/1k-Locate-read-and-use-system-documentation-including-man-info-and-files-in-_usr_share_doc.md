1.k Locate, read, and use system documentation including man, info, and files in /usr/share/doc
===

## Using System Documentation

Make sure you are familiar with the following content:

+ help flags (-h, --help, -?)
+ man
  + man -k - Search the short manual page descriptions for keywords (same as apropos)
  + man -K - Search for text in all manual pages
  + `/usr/share/man` - Location of man page files
  + mandb - create or update the manual page index caches
+ whatis - display one-line manual page descriptions
+ apropos - Search the short manual page descriptions for keywords (same as man -k)
+ info - Read documentation in Info format
  + info [command]
  + `/usr/share/info` - Location of info files
+ whereis - locate the binary, source, and manual page files for a command
+ Additional documentation for packages and binaries
  + `/usr/share/doc`

## Finding Files

**Commands:**
+ locate - Read documentation in Info format
+ find - Search for files in a directory hierarchy

### Find

Searches for files in a directory hierarchy.

Common options for `find`:
+ `-f` - files
+ `-d` - directories
+ `-l` - links
+ `-user` - File is owned by user (numeric user ID allowed)
+ `!` Is the same as `-not`


### Using 'locate'

'locate' is faster than 'find' because it has its own database (prepared by updatedb).

    # which locate
    /usr/bin/locate


**📝 NOTE:** *it looks like `locate` is installed with the systemd timer and 'updatedb'. For the test if the binary `locate` is not installed, look for `mlocate` with dnf*

Update the DB manually

    # updatedb

If you need to enable automatic update of the locate database on RHEL 8, you can do so by enabling the systemd timer

    # systemctl status mlocate-updatedb.timer  
    ● mlocate-updatedb.timer - Updates mlocate database every day
       Loaded: loaded (/usr/lib/systemd/system/mlocate-updatedb.timer; disabled; vendor preset: disabled)
       Active: inactive (dead)
      Trigger: n/a


    # systemctl enable --now mlocate-updatedb.timer  
    systemctl enable --now mlocate-updatedb.timer
    Created symlink /etc/systemd/system/timers.target.wants/mlocate-updatedb.timer → /usr/lib/systemd/system/mlocate-updatedb.timer.

    # systemctl status mlocate-updatedb.timer
    ● mlocate-updatedb.timer - Updates mlocate database every day
       Loaded: loaded (/usr/lib/systemd/system/mlocate-updatedb.timer; enabled; vendor preset: disabled)
       Active: active (waiting) since Mon 2020-12-07 15:33:02 EST; 6s ago
      Trigger: Tue 2020-12-08 00:00:00 EST; 8h left

    Dec 07 15:33:02 rhel8.localdomain systemd[1]: Started Updates mlocate database every day.

> The mlocate version published in RHEL8 does not create the /etc/cron.daily/mlocate to automatically run the updatedb daily. A systemd.timer named mlocate-updatedb.timer has replaced the cron. However, in RHEL 8 the mlocate-updatedb.timer is disabled by default.


Getting Package Related Information
---

### What package provides

Use `dnf whatprovides` to find out what package (in the repo) provides a file or package.  

#### Long listing

    # dnf whatprovides /etc/ssh/sshd_config
    Updating Subscription Management repositories.
    Last metadata expiration check: 2:37:05 ago on Thu 12 Nov 2020 01:52:19 PM EST.
    openssh-server-7.8p1-4.el8.x86_64 : An open source SSH server daemon
    Repo        : rhel-8-for-x86_64-baseos-rpms
    Matched from:
    Filename    : /etc/ssh/sshd_config

    openssh-server-8.0p1-3.el8.x86_64 : An open source SSH server daemon
    Repo        : rhel-8-for-x86_64-baseos-rpms
    Matched from:
    Filename    : /etc/ssh/sshd_config

    openssh-server-8.0p1-4.el8_1.x86_64 : An open source SSH server daemon
    Repo        : rhel-8-for-x86_64-baseos-rpms
    Matched from:
    Filename    : /etc/ssh/sshd_config

    openssh-server-8.0p1-5.el8.x86_64 : An open source SSH server daemon
    Repo        : @System
    Matched from:
    Filename    : /etc/ssh/sshd_config

    openssh-server-8.0p1-5.el8.x86_64 : An open source SSH server daemon
    Repo        : rhel-8-for-x86_64-baseos-rpms
    Matched from:
    Filename    : /etc/ssh/sshd_config

#### Short listing

    # dnf repoquery whatprovides /etc/ssh/sshd_config
    Updating Subscription Management repositories.
    Last metadata expiration check: 2:37:52 ago on Thu 12 Nov 2020 01:52:19 PM EST.
    openssh-server-0:7.8p1-4.el8.x86_64
    openssh-server-0:8.0p1-3.el8.x86_64
    openssh-server-0:8.0p1-4.el8_1.x86_64
    openssh-server-0:8.0p1-5.el8.x86_64

**📝 NOTE:** *You can use either the full path of the file/binary or a wildcard (like `*/sshd_config`)*

### Files package provides

    # rpm –ql package name

### Find manual page for package (-d)

    # rpm -qd openssh-server-8.0p1-5.el8.x86_64
    /usr/share/man/man5/moduli.5.gz
    /usr/share/man/man5/sshd_config.5.gz
    /usr/share/man/man8/sftp-server.8.gz
    /usr/share/man/man8/sshd.8.gz

---

**📌 EXAM TIP:** _If you can't remember where the docs or info files are, use `rpm` or `locate` with `grep`._

**Using `rpm`**

Find the full package name

    # rpm -qa | grep rsync
    rsync-3.1.3-9.el8.x86_64

Use `rpm -ql [package name]`

    # rpm -ql rsync-3.1.3-9.el8.x86_64 | egrep '(doc|man)'
    /usr/share/doc/rsync
    /usr/share/doc/rsync/NEWS
    /usr/share/doc/rsync/OLDNEWS
    /usr/share/doc/rsync/README
    /usr/share/doc/rsync/support
    /usr/share/doc/rsync/support/Makefile
    /usr/share/doc/rsync/support/atomic-rsync
    /usr/share/doc/rsync/support/cvs2includes
    /usr/share/doc/rsync/support/deny-rsync
    /usr/share/doc/rsync/support/file-attr-restore
    /usr/share/doc/rsync/support/files-to-excludes
    /usr/share/doc/rsync/support/git-set-file-times
    /usr/share/doc/rsync/support/instant-rsyncd
    /usr/share/doc/rsync/support/logfilter
    /usr/share/doc/rsync/support/lsh
    /usr/share/doc/rsync/support/lsh.sh
    /usr/share/doc/rsync/support/mapfrom
    /usr/share/doc/rsync/support/mapto
    /usr/share/doc/rsync/support/mnt-excl
    /usr/share/doc/rsync/support/munge-symlinks
    /usr/share/doc/rsync/support/rrsync
    /usr/share/doc/rsync/support/rsync-no-vanished
    /usr/share/doc/rsync/support/rsync-slash-strip
    /usr/share/doc/rsync/support/rsyncstats
    /usr/share/doc/rsync/support/savetransfer.c
    /usr/share/doc/rsync/tech_report.tex
    /usr/share/man/man1/rsync.1.gz


**Using `locate`**

    # locate rsync | egrep '(doc|man)'
    /usr/share/doc/rsync
    /usr/share/doc/rsync/NEWS
    /usr/share/doc/rsync/OLDNEWS
    /usr/share/doc/rsync/README
    /usr/share/doc/rsync/support
    /usr/share/doc/rsync/tech_report.tex
    /usr/share/doc/rsync/support/Makefile
    /usr/share/doc/rsync/support/atomic-rsync
    /usr/share/doc/rsync/support/cvs2includes
    /usr/share/doc/rsync/support/deny-rsync
    /usr/share/doc/rsync/support/file-attr-restore
    /usr/share/doc/rsync/support/files-to-excludes
    /usr/share/doc/rsync/support/git-set-file-times
    /usr/share/doc/rsync/support/instant-rsyncd
    /usr/share/doc/rsync/support/logfilter
    /usr/share/doc/rsync/support/lsh
    /usr/share/doc/rsync/support/lsh.sh
    /usr/share/doc/rsync/support/mapfrom
    /usr/share/doc/rsync/support/mapto
    /usr/share/doc/rsync/support/mnt-excl
    /usr/share/doc/rsync/support/munge-symlinks
    /usr/share/doc/rsync/support/rrsync
    /usr/share/doc/rsync/support/rsync-no-vanished
    /usr/share/doc/rsync/support/rsync-slash-strip
    /usr/share/doc/rsync/support/rsyncstats
    /usr/share/doc/rsync/support/savetransfer.c
    /usr/share/man/man1/rsync.1.gz

---
[⬅️ Back](1-Understand-and-use-essential-tools.md)
