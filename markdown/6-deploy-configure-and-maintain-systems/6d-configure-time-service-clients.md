# 6.d Configure time service clients

## NTP Overview

The Network Time Protocol (NTP) is a networking protocol for clock synchronization between computer systems over packet-switched, variable-latency data networks.

NTP synchronizes the clocks of computer systems. It can read the time from a reference source, then transmit the reading to one or more clients and adjust each client clock as required

### Stratum

Large computer networks can have many devices acting as NTP servers. Where multiple NTP sources are available, NTP clients need some way of (a) judging which time sources are likely to be the most accurate, and (b) preventing timing loops*. The NTP protocol achieves these aims by including a simple measure of the synchronisation distance from the primary time source. This is known as the Stratum level.

Stratum 1 indicates a computer that has a true real-time reference directly connected to it (e.g. GPS, atomic clock, etc.), such computers are expected to be very close to real time. Stratum 2 computers are those which have a stratum 1 server; stratum 3 computers have a stratum 2 server and so on. A value of 10 indicates that the clock is so many hops away from a reference clock that its time is fairly unreliable.

![](6d-configure-time-service-clients/6d-configure-time-service-clients-3f5d6.png)

### Servers and clients

An NTP server is a source of time information, and an NTP client is a system/device that is attempting to synchronize its clock to a server. Servers can be either a primary or secondary server

+ A primary server receives UTC time signals directly from a very accurate source such as an atomic clock or more commonly - a GPS signal source.A secondary server receives its time signal from one or more upstream servers, and distributes its time signals to one or more downstream servers and clients.
+ Secondary servers are arranged in a strict hierarchy in terms of upstream and downstream, and the stratum terminology is often used to assist in this process.

### 127.127.1.0

127.127.1.0 is an ntpd-specific way to enable the local refclock driver (note that this is not supported in chrony and it's only here as Reference).


## Data and Time Commands

### date

Display the current time in the given FORMAT, or set the system date.

    $ date
    Thu Dec 10 13:58:14 EST 2020

### timedatectl

Use `timedatectl` to manage time and timezones.  

Main commands:
- `status` - Show current settings of the system clock and RTC, including whether network time synchronization through systemd-timesyncd.service is active
- `show` - Show the same information as status, but in machine readable form
- `set-time [TIME]` - Set the system clock to the specified time
- `set-timezone [TIMEZONE]` - Set the system time zone to the specified value
- `list-timezones` - List available time zones, one per line
- `set-ntp [BOOL]` - Enables/disables NTP

      # timedatectl  
                     Local time: Mon 2020-11-23 17:27:11 EST
                 Universal time: Mon 2020-11-23 22:27:11 UTC
                       RTC time: Mon 2020-11-23 22:27:11
                      Time zone: America/Toronto (EST, -0500)
      System clock synchronized: no
                    NTP service: active
                RTC in local TZ: no

**📝NOTE:** _To use NTP the 'chrony' service needs to be running. You may also need to restart the service after enabling NTP with `timedatectl`._

### tzselect

You can also use '`tzselect`' to select the timezone. This script will ask you questions through multiple menus.  

    # tzselect  
    Please identify a location so that time zone rules can be set correctly.
    Please select a continent, ocean, "coord", or "TZ".
    1) Africa
    2) Americas
    3) Antarctica
    4) Asia
    5) Atlantic Ocean
    6) Australia
    7) Europe
    8) Indian Ocean
    9) Pacific Ocean
    10) coord - I want to use geographical coordinates.
    11) TZ - I want to specify the time zone using the Posix TZ format.
    #?  

### hwclock

`hwclock` is an administration tool for the time clocks.  

`hwclock` can:
- display the Hardware Clock time
- set the Hardware Clock to a specified time
- set the Hardware Clock from the System Clock
- set the System Clock from the Hardware Clock
- compensate for Hardware Clock drift
- correct the System Clock timescale
- set the kernel's timezone, NTP timescale, and epoch (Alpha only)
- predict future Hardware Clock values based on its drift rate

### Chrony

The Chrony daemon, chronyd, runs in the background and monitors the time and status of the time server specified in the `chrony.conf` file. If the local time needs to be adjusted, chronyd does it smoothly without the programmatic trauma that would occur if the clock were instantly reset to a new time.

Configuration for chronyd can be made by editing `/etc/chrony.conf`.

> **chronyd** - chronyd is a daemon for synchronisation of the system clock. It can synchronise the clockwith NTP servers, reference clocks (e.g. a GPS receiver), and manual input using wristwatchand keyboard via chronyc.

Chrony's `chronyc` tool allows someone to monitor the current status of Chrony and make changes if necessary. The `chronyc` utility can be used as a command that accepts subcommands, or it can be used as an interactive text-mode program.

> **chronyc** - chronyc is a command-line interface program which can be used to monitor chronyd’s performance and to change various operating parameters whilst it is running

#### Configuring chrony to use a NTP Server

Edit '/etc/chrony.conf' and comment out the line starting with 'pool'

    # pool 2.rhel.pool.ntp.org iburst

Add a line with 'server [hostname|ip]' after pool

    # pool 2.rhel.pool.ntp.org iburst
    server 10.13.17.2

Restart chronyd

    # systemctl restart chronyd.service

Use 'chronyc sources' to check that it's trying to connect to the new server

    # chronyc sources
    210 Number of sources = 4
    MS Name/IP address         Stratum Poll Reach LastRx Last sample                
    ===============================================================================
    ^- 10.13.17.2          2   6   377    41  +1987us[+1977us] +/-   41ms

#### Syncing Chrony to the local clock

The `local` keyword is used to allow chronyd to appear synchronized to real time from the viewpoint of clients polling it, even if it has no current synchronization source. This option is normally used on the "master" computer in an isolated network, where several computers are required to synchronize to one another, and the "master" is kept in line with real time by manual input.

##### Example isolated network configuration

**_Master_**

`/etc/chrony.conf`

    driftfile /var/lib/chrony/drift
    commandkey 1
    keyfile /etc/chrony.keys
    initstepslew 10 client1 client3 client6
    local stratum 8
    manual
    allow 192.0.2.0

_Where 192.0.2.0 is the network or subnet address from which the clients are allowed to connect._

**_Slave_**

`/etc/chrony.conf`

    server master
    driftfile /var/lib/chrony/drift
    logdir /var/log/chrony
    log measurements statistics tracking
    keyfile /etc/chrony.keys
    commandkey 24
    local stratum 10
    initstepslew 20 master
    allow 192.0.2.123

_Where 192.0.2.123 is the address of the master._


#### chronyc - Common Options

The chronyc command, when used with the tracking subcommand, provides statistics that report how far off the local system is from the reference server.

    # chronyc tracking
    Reference ID    : C06302AC (draco.spiderspace.co.uk)
    Stratum         : 3
    Ref time (UTC)  : Wed Mar 11 14:24:19 2020
    System time     : 0.000012882 seconds slow of NTP time
    Last offset     : -0.000013424 seconds
    RMS offset      : 0.000063391 seconds
    Frequency       : 17.800 ppm slow
    Residual freq   : -0.004 ppm
    Skew            : 0.380 ppm
    Root delay      : 0.008013672 seconds
    Root dispersion : 0.003327465 seconds
    Update interval : 128.2 seconds
    Leap status     : Normal

The sources subcommand is also useful because it provides information about the time source configured in chrony.conf

    # chronyc sources
    210 Number of sources = 4
    MS Name/IP address         Stratum Poll Reach LastRx Last sample                
    ===============================================================================
    ^- dc1-recdns02.bellmtsdata>     2   7   377   114   -109us[ -109us] +/-   56ms
    ^- 154.11.146.39                 2   7   377   115  -2050us[-2050us] +/-   66ms
    ^* draco.spiderspace.co.uk       2   7   377   118    -93us[ -112us] +/- 7960us
    ^- ns2.switch.ca                 2   7   377   115  +3974us[+3974us] +/-   50ms

You can also get a legend with the `-v` flag

    # chronyc sources -v
    210 Number of sources = 8

      .-- Source mode  '^' = server, '=' = peer, '#' = local clock.
     / .- Source state '*' = current synced, '+' = combined , '-' = not combined,
    | /   '?' = unreachable, 'x' = time may be in error, '~' = time too variable.
    ||                                                 .- xxxx [ yyyy ] +/- zzzz
    ||      Reachability register (octal) -.           |  xxxx = adjusted offset,
    ||      Log2(Polling interval) --.      |          |  yyyy = measured offset,
    ||                                \     |          |  zzzz = estimated error.
    ||                                 |    |           \
    MS Name/IP address         Stratum Poll Reach LastRx Last sample               
    ===============================================================================
    ^? dc1-recdns02.bellmtsdata>     0   7     0     -     +0ns[   +0ns] +/-    0ns
    ^* time.cloudflare.com           3   6    17    58   -690us[ +533us] +/-   20ms
    ^+ time.cloudflare.com           3   6    17    58   +469us[+1294us] +/-   19ms
    ^+ ntp.nyy.ca                    1   6    17    58   +830us[+1878us] +/-   34ms
    ^? time.cloudflare.com           0   6     0     -     +0ns[   +0ns] +/-    0ns
    ^? 2001:470:b2de::1              0   6     0     -     +0ns[   +0ns] +/-    0ns
    ^? time.cloudflare.com           0   6     0     -     +0ns[   +0ns] +/-    0ns
    ^? bgp-router00-van.van.the>     0   6     0     -     +0ns[   +0ns] +/-    0ns

---

#### Additional Info

**See man pages for:**
- chronyd (8)            - chrony daemon
- chrony.conf (5)      - chronyd configuration file
- chronyc (1)             - command-line interface for chrony daemon
- timedatectl (1)      - Control the system time and date
- tzselect (8)             - select a timezone
---
[⬅️ Back](6-deploy-configure-and-maintain-systems.md)
