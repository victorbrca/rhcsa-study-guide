# 6.a Schedule tasks using at and cron

**⚠️ WARNING:** _For tasks to work, it's important that the user credentials have not expired. Use 'chage -l [user]' to check._

    [root@rhel8 system]# chage -l root
    Last password change					: Nov 11, 2020
    Password expires					: never
    Password inactive					: never
    Account expires						: never
    Minimum number of days between password change		: 0
    Maximum number of days between password change		: 99999
    Number of days of warning before password expires	: 7


## Scheduled Tasks with 'at'

**Commands:**
- at (1)                    - queue, examine or delete jobs for later execution
- atq                       - Lists the user's pending jobs (part of 'at')
- atrm (1)             - queue, examine or delete jobs for later execution
- at.allow (5)         - determine who can submit jobs via at or batch
- at.deny (5)          - determine who can submit jobs via at or batch

### Getting Started

You might need to install 'at'

    # dnf install -y at

Start and enable the service

    # systemctl enable --now atd.service

### Usage

You can feed at:
- a command
- file with commands
- script

When you run 'at' by itself (with a timespec) it will prompt you for the commands with an interactive prompt. Use 'Ctrl+d' to exit.  

**Common commands:**
- at now +1 minute  - Run the job in 1 minute
- at 12:00am              - Run at 12am
- atq                            - Shows the at job queue
- atrm [job number] - Removes the job number
- at -c [job number]  - Shows the command the job is running

#### User Control for 'at'  

at.allow, at.deny - determine who can submit jobs via at or batch
The /etc/at.allow and /etc/at.deny files determine which user can submit commands for later execution via 'at' or 'batch'

## Cron

**Commands:**
- cron (8)             - daemon to execute scheduled commands
- cronnext (1)     - time of next job cron will execute

### Understanding the Difference Between Cron, Crontab and Anacron

**Cron/crond** - daemon to execute scheduled commands

**Crontab** - Crontab is the program used to install a crontab table file, remove or list the existing tables used to serve the cron(8) daemon

**Anacron** - Anacron is used to execute commands periodically, with a frequency specified in days.  Unlike cron, it does not assume that the machine is running continuously.  Hence, it can be used on machines  that  are  not running  24 hours a day to control regular jobs as daily, weekly, and monthly jobs.

Cron checks these files and directories:  
- **/etc/crontab** - System crontab.  Nowadays the file is empty by default. Originally it was usually used to run daily, weekly, monthly jobs. By default these jobs are now run through anacron which reads /etc/anacrontab configuration file.  See anacrontab(5) for more details.
- **/etc/cron.d/** - Directory that contains system cronjobs stored for different users.
- **/var/spool/cron** - Directory that contains user crontables created by the crontab command.
- **/etc/cron.hourly** - This is where cron executed Anacron from

      cron (crond.service)
              ├─> /etc/crond.hourly/*
              │                     └─> 0anacron
              │                             └─> /usr/sbin/anacron -s
              │                                          └─> /etc/anacrontab
              │                                                     ├─> run-parts -> /etc/cron.daily/*
              │                                                     ├─> run-parts -> /etc/cron.weekly/*
              │                                                     └─> run-parts -> /etc/cron.monthly/*
              │
              ├─> /etc/crontab (system crontab - obsolete)
              │
              ├─> /etc/cron.d/* (system cronjobs for users)
              │
              └─> /var/spool/cron/ (user crontabs)
                         ├─> user1
                         └─> user2

### Crontab

**Commands:**
- crontab (1)          - maintains crontab files for individual users

**📝 NOTES**
- Variables (like PATH, SHELL and MAILTO) can be setup in the crontab file
- Jobs that are not user specific are usually added to '/etc/cron.d' and they usually include the user name

Example crontab template

    #SHELL=/bin/bash
    #PATH=/sbin:/bin:/usr/sbin:/usr/bin
    #MAILTO=root
    #===============================================================================
    # +--------- Minute (0-59)                    | Output Dumper: >/dev/null 2>&1
    # | +------- Hour (0-23)                      | Multiple Values Use Commas: 3,12,47
    # | | +----- Day Of Month (1-31)              | Do every X intervals: */X  -> Example: */15 * * * *  Is every 15 minutes
    # | | | +--- Month (1 -12 or Jan-Dec)         | Aliases: @reboot -> Run once at startup; @hourly -> 0 * * * *;
    # | | | | +- Day Of Week (0-6) (Sunday = 0)   | @daily -> 0 0 * * *; @weekly -> 0 0 * * 0; @monthly ->0 0 1 * *;
    # | | | | |                                   | @yearly -> 0 0 1 1 *;
    # * * * * * COMMAND                           |
    #===============================================================================

To show the contents of a the crontab without editing

    # crontab -l  

Edit crontab

    # crontab -e

To edit the crontab of another user

    # crontab -u [user] -e  

To list the crontab of another user

    # crontab -u [user] -l

#### User Control for 'crontab'  

You can use '/etc/cron.deny' and '/etc/cron.allow'

### Anacron

**Commands:**
- anacron (8)          - runs commands periodically
- anacrontab (5)       - configuration file for Anacron

Anacron  is  used  to  execute  commands  periodically, with a frequency specified in days. Unlike cron(8), it does not assume that the machine is running continuously.  Hence, it can be used on machines that are not running 24 hours a day to control regular jobs as daily, weekly, and monthly jobs.

Anacron reads a list of jobs from the `/etc/anacrontab` configuration file (see anacrontab(5)). This file contains the list of jobs that Anacron controls. Each job entry specifies a period in days, a delay in minutes, a unique job identifier, and a shell command

    # cat /etc/anacrontab  
    # /etc/anacrontab: configuration file for anacron

    # See anacron(8) and anacrontab(5) for details.

    SHELL=/bin/sh
    PATH=/sbin:/bin:/usr/sbin:/usr/bin
    MAILTO=root
    # the maximal random delay added to the base delay of the jobs
    RANDOM_DELAY=45
    # the jobs will be started during the following hours only
    START_HOURS_RANGE=3-22

    #period in days   delay in minutes   job-identifier   command
    1    5    cron.daily        nice run-parts /etc/cron.daily
    7    25    cron.weekly        nice run-parts /etc/cron.weekly
    @monthly 45    cron.monthly        nice run-parts /etc/cron.monthly

Run status:

    # ll /var/spool/anacron/
    total 12
    -rw-------. 1 root root 9 Mar 11 14:28 cron.daily
    -rw-------. 1 root root 9 Mar 10 18:02 cron.monthly
    -rw-------. 1 root root 9 Mar 10 17:42 cron.weekly

#### Disabling Anacron

In case you want to disable Anacron, add a line with `0anacron` which is the name of the script running the Anacron into the /etc/cron.hourly/jobs.deny file.

---
[⬅️ Back](6-deploy-configure-and-maintain-systems.md)
