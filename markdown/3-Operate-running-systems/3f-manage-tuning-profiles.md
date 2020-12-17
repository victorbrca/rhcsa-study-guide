# 3.f Manage tuning profiles

[RHEL 8 > Monitoring and managing system status and performance> Chapter 2. Getting started with tuned](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/monitoring_and_managing_system_status_and_performance/getting-started-with-tuned_monitoring-and-managing-system-status-and-performance)

Tuned is a service that monitors your system and optimizes the performance under certain workloads. The core of Tuned are profiles, which tune your system for different use cases.     

## Tuned profiles

A detailed analysis of a system can be very time-consuming. Tuned provides a number of predefined profiles for typical use cases. You can also create, modify, and delete profiles.  

The profiles provided with Tuned are divided into the following categories:  
+ Power-saving profiles  
+ Performance-boosting profiles       

## Pre-setup to Use Tuned

Make sure that the package is installed with one of the commands below:

    [root@localhost systemd]# rpm -qa | grep tuned
    tuned-2.14.0-3.el8.noarch

    [root@localhost systemd]# command -v tuned
    /usr/sbin/tuned

    [root@localhost systemd]# command -v tuned-adm
    /usr/sbin/tuned-adm

Make sure the systemd service is running

    # systemctl status tuned.service  
    ● tuned.service - Dynamic System Tuning Daemon
       Loaded: loaded (/usr/lib/systemd/system/tuned.service; enabled; vendor preset: enabled)
       Active: active (running) since Sat 2020-11-14 14:10:34 EST; 2 days ago
         Docs: man:tuned(8)
               man:tuned.conf(5)
               man:tuned-adm(8)
    Main PID: 1030 (tuned)
        Tasks: 4 (limit: 12285)
       Memory: 17.1M
       CGroup: /system.slice/tuned.service
               └─1030 /usr/libexec/platform-python -Es /usr/sbin/tuned -l –P

## Using Tuned  

### Getting Info

List Profiles

    # tuned-adm list
    Available profiles:
    - accelerator-performance     - Throughput performance based tuning with disabled higher latency STOP states
    - balanced                    - General non-specialized tuned profile
    - desktop                     - Optimize for the desktop use-case
    - hpc-compute                 - Optimize for HPC compute workloads
    - intel-sst                   - Configure for Intel Speed Select Base Frequency
    - latency-performance         - Optimize for deterministic performance at the cost of increased power consumption
    - network-latency             - Optimize for deterministic performance at the cost of increased power consumption, focused on low latency network performance
    - network-throughput          - Optimize for streaming network throughput, generally only necessary on older CPUs or 40G+ networks
    - optimize-serial-console     - Optimize for serial console use.
    - powersave                   - Optimize for low power consumption
    - throughput-performance      - Broadly applicable tuning that provides excellent performance across a variety of common server workloads
    - virtual-guest               - Optimize for running inside a virtual guest
    - virtual-host                - Optimize for running KVM guests
    Current active profile: virtual-guest

Show current active profile

    # tuned-adm active
    Current active profile: virtual-guest

Verifies current profile against system settings

    # tuned-adm verify
    Verfication succeeded, current system settings match the preset profile.
    See tuned log file ('/var/log/tuned/tuned.log') for details.

Recommend a profile suitable for your system

    # tuned-adm recommend  
    virtual-guest

Get information on the profile

    # tuned-adm profile_info virtual-guest
    Profile name:
    virtual-guest

    Profile summary:
    Optimize for running inside a virtual guest

    Profile description:

### Switching Profiles

Switches to the given profile

    # tuned-adm profile powersave

    # tuned-adm active
    Current active profile: powersave


### Disabling Tuned

#### Temporarily

    # tuned-adm off

#### At Boot

Use systemd

    # systemctl disable --now tuned


---
[⬅️ Back](3-Operate-running-systems.md)
