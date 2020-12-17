# 6.e Install and update software packages from Red Hat Network, a remote repository, or from the local file system

## Red Hat Package Manager (RPM)

Originally how to install software on the RHEL servers, and it's still a way to manage software on the server.
Flags:
- i- Install
- U - Upgrade
- h - Progress
- v - Verbose
- e - Remove
- nodeps - Ignore dependencies
- force - Ignore errors

## Yellowdog Updater Modified (YUM) v4 / DNF

**📝NOTE:** _On Red Hat Enterprise Linux (RHEL) 8, installing software is ensured by the new version of the YUM tool, which is based on the DNF technology (YUM v4). The file '/bin/yum' is a symbolic link to '/bin/dnf'_  

YUM configuration is in '/etc/yum.conf -> dnf/dnf.conf'.  

### Useful Commands

    # dnf  
    check-update  groupinfo     groupupdate   list          provides      search        whatprovides
    clean         groupinstall  help          localinstall  remove        shell          
    deplist       grouplist     info          localupdate   repolist      update         
    erase         groupremove   install       makecache     resolvedep    upgrade   

#### Search/Get Info on Packages

Search for a package (by name and description) on enabled repos

    # dnf search chrony
    Updating Subscription Management repositories.
    Last metadata expiration check: 2:02:27 ago on Mon 23 Nov 2020 03:55:56 PM EST.
    ===================================== Name Exactly Matched: chrony ======================================
    chrony.x86_64 : An NTP client/server

Lists packages that match name on enabled repos or RPMDB (RPM database)

    # dnf list chrony
    Updating Subscription Management repositories.
    Last metadata expiration check: 2:07:31 ago on Mon 23 Nov 2020 03:55:56 PM EST.
    Installed Packages
    chrony.x86_64                                     3.5-1.el8                                     @anaconda

Gets information about a package

    # dnf info chrony
    Updating Subscription Management repositories.
    Last metadata expiration check: 2:07:06 ago on Mon 23 Nov 2020 03:55:56 PM EST.
    Installed Packages
    Name         : chrony
    Version      : 3.5
    Release      : 1.el8
    Architecture : x86_64
    Size         : 634 k
    Source       : chrony-3.5-1.el8.src.rpm
    Repository   : @System
    From repo    : anaconda
    Summary      : An NTP client/server
    URL          : https://chrony.tuxfamily.org
    License      : GPLv2
    Description  : chrony is a versatile implementation of the Network Time Protocol (NTP).
                 : It can synchronise the system clock with NTP servers, reference clocks
                 : (e.g. GPS receiver), and manual input using wristwatch and keyboard. It
                 : can also operate as an NTPv4 (RFC 5905) server and peer to provide a time
                 : service to other computers in the network.

#### Install

dnf can also be used to install local rpm (instead of the 'rpm' command), and it will automatically check for deps and install them.  

    # dnf install ~/Downloads/tito-0.6.2-1.fc22.noarch.rpm

You can use the 'install' command to install a repo.

    # dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

Install a group

    # dnf group install file-server

#### Remove

Removes a package

    # dnf remove chrony

Removes a group

    # dnf group remove file-server

#### History

The history command allows the user to view what has happened in past transactions and act according to this information.
Show the history of transactions

    # dnf history
    Updating Subscription Management repositories.
    ID     | Command line                                       | Date and time    | Action(s)      | Altered
    ---------------------------------------------------------------------------------------------------------
         8 | install -y stratis-cli stratisd                    | 2020-11-21 20:52 | Install        |   11    
         7 | install -y autofs                                  | 2020-11-20 17:48 | Install        |    1    
         6 | install -y tuna                                    | 2020-11-16 16:32 | Install        |    1    
         5 | update -y                                          | 2020-11-14 13:41 | Upgrade        |    1    
         4 | install kernel-devel elfutils-libelf-devel         | 2020-11-11 13:37 | Install        |    2    
         3 | groupinstall Development Tools                     | 2020-11-11 13:37 | Install        |    1    
         2 | update                                             | 2020-11-11 13:09 | I, U           |   35 EE
         1 |                                                    | 2020-11-11 12:15 | Install        | 1491 EE

Redo the last transaction (repeats the specified transaction)

    # dnf history redo [0-9]

Rollback the last transaction (undo all transactions performed after the specified transaction)

    # dnf history rollback [0-9]

Undo the last transaction (removes all changes performed by the specified transaction)

    # dnf history undo [0-9]

### Working with Repos

Repos are configured in '/etc/yum.repos.d'.  

List repos  

    # dnf repolist  
    Updating Subscription Management repositories.
    repo id                                  repo name
    rhel-8-for-x86_64-appstream-rpms         Red Hat Enterprise Linux 8 for x86_64 - AppStream (RPMs)
    rhel-8-for-x86_64-baseos-rpms            Red Hat Enterprise Linux 8 for x86_64 - BaseOS (RPMs)

Get information about repos

    # dnf repoinfo
    Updating Subscription Management repositories.
    Last metadata expiration check: 1:55:31 ago on Mon 23 Nov 2020 03:55:56 PM EST.
    Repo-id            : rhel-8-for-x86_64-appstream-rpms
    Repo-name          : Red Hat Enterprise Linux 8 for x86_64 - AppStream (RPMs)
    Repo-revision      : 1605729343
    Repo-updated       : Wed 18 Nov 2020 02:55:42 PM EST
    Repo-pkgs          : 15,106
    Repo-available-pkgs: 13,502
    Repo-size          : 33 G
    Repo-baseurl       : https://cdn.redhat.com/content/dist/rhel8/8/x86_64/appstream/os
    Repo-expire        : 86,400 second(s) (last: Mon 23 Nov 2020 03:55:56 PM EST)
    Repo-filename      : /etc/yum.repos.d/redhat.repo
    Repo-id            : rhel-8-for-x86_64-baseos-rpms
    Repo-name          : Red Hat Enterprise Linux 8 for x86_64 - BaseOS (RPMs)
    Repo-revision      : 1605087736
    Repo-updated       : Wed 11 Nov 2020 04:42:14 AM EST
    Repo-pkgs          : 6,129
    Repo-available-pkgs: 6,126
    Repo-size          : 8.1 G
    Repo-baseurl       : https://cdn.redhat.com/content/dist/rhel8/8/x86_64/baseos/os
    Repo-expire        : 86,400 second(s) (last: Mon 23 Nov 2020 03:55:56 PM EST)
    Repo-filename      : /etc/yum.repos.d/redhat.repo
    Total packages: 21,235

#### Manually Adding a Repo

Browse to '/etc/yum.repos.d'

    # cd /etc/yum.repos.d

Create a new file with extension of '.repo'. You can use the existing files as templates

    [repo name]
    name=[description of the repo]
    baseurl=[path file:///path/to/reo/; url http://url; ftp ftp://url; for the repo]
    enabled=[1=yes, 0=no]

#### Using dnf config-manager

**📝 NOTE:** *You can also use `yum-config-manager`, however it's not installed by default. You will need to install either `dnf-utils` or `yum-utils`.*

Add a repository  

    # dnf config-manager --add-repo [repo url]

Enable a repo

    # dnf config-manager --enable [repo name]

Disable a repo

    # dnf config-manager --disable [repo name]

#### Creating a Repo with 'createrepo'

Install 'createrepo'  

    # dnf install -y createrepo

Create a folder  

    # mkdir /root/my_repo  

Add all the binaries that you would like to have on that repo

    # dnf install -y --downloadonly --downloaddir=/root/my_repo nmap httpd mysql

Use 'createrepo' to create a repo database

    # createrepo --database /root/my_repo/

Add the repo

    # dnf config-manager --add-repo file:///root/my_repo/
    Updating Subscription Management repositories.
    Adding repo from: file:///root/my_repo/

Check that repo was added

    # dnf repolist
    Updating Subscription Management repositories.
    repo id                                  repo name
    rhel-8-for-x86_64-appstream-rpms         Red Hat Enterprise Linux 8 for x86_64 - AppStream (RPMs)
    rhel-8-for-x86_64-baseos-rpms            Red Hat Enterprise Linux 8 for x86_64 - BaseOS (RPMs)
    root_my_repo                            created by dnf config-manager from file:///root/my_repo/

Check that you can see the files

    # dnf repository-packages root_my_repo list

Disable the repo

    # dnf config-manager --disable root_my_repo_
    Updating Subscription Management repositories.

---

#### Commands and Programs
- dnf repolist                - Depending  on  the  exact  command lists enabled, disabled or all known repositories
- dnf-config-manager (8) - DNF config-manager Plugin
- createrepo_c.x86_64 : Creates a common metadata repository

---
[⬅️ Back](6-deploy-configure-and-maintain-systems.md)
