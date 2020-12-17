8.b Change passwords and adjust password aging for local user accounts
===

Commands
---

### passwd

Change users password. Only root can change the password for another user.  

### chage

chage - Change user password expiry information

Common options:
+ `d` - Set date of last password change to `LAST_DAY`
+ `E` - Set the account to expire on date
+ `M` - Maximum days the password will be valid for
+ `I` - Number of days of inactivity after expiration where account will be locked
+ `W` - Number of days to warn that password is expiring

**📝 NOTE:** *Having an expired password doesn't mean that the account is locked. It means that the user can still login, but is prompted to change the password*

#### Examples

List users accounts aging information

    # chage -l user1
    Last password change                    : Nov 26, 2020
    Password expires                    : never
    Password inactive                    : never
    Account expires                        : never
    Minimum number of days between password change        : 0
    Maximum number of days between password change        : 99999
    Number of days of warning before password expires    : 7

Force user to change password on next login

    # chage -d 0 user1

Set account to expire on December 31st 2020

    # chage -E 2020-12-31 user1

    # chage -l user1 | grep 'Account expires'
    Account expires                        : Dec 31, 2020

Remove account expiration

    # chage -E -1 user1

    # chage -l user1 | grep 'Account expires'
    Account expires                        : never

Set the password to expire in 30 days

    # chage -M 30 user1

    # chage -l user1 | grep  'Password expires'
    Password expires                    : Dec 26, 2020

Remove password expiration

    # chage -M -1 user1  

    # chage -l user1 | grep  'Password expires'
    Password expires                    : never

**Interactive Mode**

You can also run 'chage' in interactive mode by calling it with a username and not other arguments.

    # chage user1
    Changing the aging information for user1
    Enter the new value, or press ENTER for the default

        Minimum Password Age [0]: 0
        Maximum Password Age [-1]:  
        Last Password Change (YYYY-MM-DD) [2020-11-26]:  
        Password Expiration Warning [7]:  
        Password Inactive [-1]: 1
        Account Expiration Date (YYYY-MM-DD) [-1]:  


### Configuring Defaults

Default password age and requirements configuration can be made in '/etc/login.defs'

    # Password aging controls:
    #
    #       PASS_MAX_DAYS   Maximum number of days a password may be used.
    #       PASS_MIN_DAYS   Minimum number of days allowed between password changes.
    #       PASS_MIN_LEN    Minimum acceptable password length.
    #       PASS_WARN_AGE   Number of days warning given before a password expires.
    #
    PASS_MAX_DAYS   99999
    PASS_MIN_DAYS   0
    PASS_MIN_LEN    5
    PASS_WARN_AGE   7

Additional password configuration, like inactivity and expiration date, can be set in `/etc/default/useradd`.

By default, `/etc/default/useradd` usually looks like this:

    # useradd defaults file
    GROUP=100
    HOME=/home
    INACTIVE=-1
    EXPIRE=
    SHELL=/bin/bash
    SKEL=/etc/skel
    CREATE_MAIL_SPOOL=yes

Edit the `INACTIVE` line and add the `EXPIRE` line if needed:

    INACTIVE=3         # Expires after 3 days of inactivity
    EXPIRE=2020-12-31  # Expires on Dec 31 2020

Password Complexity
---

Password complexity can be achieved with 'pam_pwquality.so'.

**Man page:**
+ pam_pwquality (8)    - PAM module to perform password quality checking

**📝NOTE:** *Understanding and managing pam is not part of RHCSA exam.*

---
[⬅️ Back](8-Manage-users-and-groups.md)
