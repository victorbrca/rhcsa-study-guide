# 3.d Identify CPU/memory intensive processes, adjust process priority with renice, and kill processes

## top

The top program provides a dynamic real-time view of a running system. It can display system summary information as well as a list of processes or threads currently being  managed by the Linux kernel.

    top - 12:32:15 up 11 days, 17:00,  2 users,  load average: 0.01, 0.03, 0.00
    Tasks: 131 total,   1 running, 130 sleeping,   0 stopped,   0 zombie
    %Cpu(s):  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
    MiB Mem :    821.2 total,    185.4 free,    216.3 used,    419.5 buff/cache
    MiB Swap:   2116.0 total,   2095.9 free,     20.1 used.    465.3 avail Mem

      PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND                             
        1 root      20   0  179176  13480   9140 S   0.0   1.6   0:11.96 systemd                             
        2 root      20   0       0      0      0 S   0.0   0.0   0:00.18 kthreadd                            
        3 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 rcu_gp                              
        4 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 rcu_par_gp                          
        6 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/0:0H-kblockd                
        8 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 mm_percpu_wq                        
        9 root      20   0       0      0      0 S   0.0   0.0   0:06.55 ksoftirqd/0                         
       10 root      20   0       0      0      0 I   0.0   0.0   0:21.09 rcu_sched                           
       11 root      rt   0       0      0      0 S   0.0   0.0   0:00.03 migration/0                         
       12 root      rt   0       0      0      0 S   0.0   0.0   0:00.14 watchdog/0                          
       13 root      20   0       0      0      0 S   0.0   0.0   0:00.00 cpuhp/0                             
       14 root      20   0       0      0      0 S   0.0   0.0   0:00.00 cpuhp/1                             
       15 root      rt   0       0      0      0 S   0.0   0.0   0:00.68 watchdog/1                          
       16 root      rt   0       0      0      0 S   0.0   0.0   0:00.02 migration/1                         
       17 root      20   0       0      0      0 S   0.0   0.0   0:00.34 ksoftirqd/1                         
       19 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/1:0H-kblockd                
       21 root      20   0       0      0      0 S   0.0   0.0   0:00.00 kdevtmpfs                           
       22 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 netns                               
       23 root      20   0       0      0      0 S   0.0   0.0   0:00.03 kauditd                             
       25 root      20   0       0      0      0 S   0.0   0.0   0:00.21 khungtaskd

Remember that `top` can display:
+ Load average
+ Number of tasks
+ Information about specific tasks (PID, nice, CPU%, MEM%, System time, etc..)

**Important Keyboard shortcuts:**
+ `t` - change CPU display
+ `m` - Change memory display
+ `f` - Select sorting field
+ `<` `>` - Walk through sorting field
+ `u` - Filter by user
+ `r` - Renice task
+ `k` - Kill task
+ `R` - Reverse sort
+ `c` - Command/line
+ `1` - CPUs

## ps

`ps` displays information about a selection of the active processes.

    # ps
      PID TTY          TIME CMD
    27278 pts/1    00:00:00 bash
    27385 pts/1    00:00:00 ps

    # ps -ef | grep rsyslogd
    root      1162     1  0 Nov28 ?        00:00:41 /usr/sbin/rsyslogd -n
    root     27501 27278  0 12:42 pts/1    00:00:00 grep --color=auto rsyslogd

This version of ps accepts several kinds of options:
+ 1 - UNIX options, which may be grouped and must be preceded by a dash.
+ 2 - BSD options, which may be grouped and must not be used with a dash.
+ 3 - GNU long options, which are preceded by two dashes.

**Important Options:**
+ a- all user's processes which are attached to a terminal
+ u - display in user format
+ x - lists all the invoking user's processes

> Note that `ps -aux` is distinct from `ps aux`.  The POSIX and UNIX standards require that `ps -aux` print all processes owned by a user named "x", as well as printing all processes that would be selected by the `-a` option.  If the user named "x" does not exist, this ps may interpret the command as `ps aux` instead and print a warning.  This behavior is intended to aid in transitioning old scripts and habits. It is fragile, subject to change, and thus should not be relied upon.

## kill (signals)

The command kill sends the specified signal to the specified processes or process groups. If  no  signal is specified, the TERM signal is sent.

+ SIGTERM (15) - Asks the process to exit cleanly (the default signal used with kill)
+ SIGKILL (9) - Stops the process immediately (dirty)
+ SIGHUP (1) - Stops process in a shell environment. Can make some services re-read configuration files
+ SIGINT (2) - Same as Ctrl+C
+ SIGCONT (18) - Starts a process that was paused with SIGSTOP or SIGTSTP
+ SIGSTOP (19) - (Ctrl+x) Pauses a process so it can be started later (does not rely on binary)
+ SIGTSTP (20) - Same as Ctrl-z, but relies on binary to know what to do

**📌 EXAM TIP:** *Use `kill -l` to list all the kill signals.*

    # kill -l
     1) SIGHUP	 2) SIGINT	 3) SIGQUIT	 4) SIGILL	 5) SIGTRAP
     6) SIGABRT	 7) SIGBUS	 8) SIGFPE	 9) SIGKILL	10) SIGUSR1
    11) SIGSEGV	12) SIGUSR2	13) SIGPIPE	14) SIGALRM	15) SIGTERM
    16) SIGSTKFLT	17) SIGCHLD	18) SIGCONT	19) SIGSTOP	20) SIGTSTP
    21) SIGTTIN	22) SIGTTOU	23) SIGURG	24) SIGXCPU	25) SIGXFSZ
    26) SIGVTALRM	27) SIGPROF	28) SIGWINCH	29) SIGIO	30) SIGPWR
    31) SIGSYS	34) SIGRTMIN	35) SIGRTMIN+1	36) SIGRTMIN+2	37) SIGRTMIN+3
    38) SIGRTMIN+4	39) SIGRTMIN+5	40) SIGRTMIN+6	41) SIGRTMIN+7	42) SIGRTMIN+8
    43) SIGRTMIN+9	44) SIGRTMIN+10	45) SIGRTMIN+11	46) SIGRTMIN+12	47) SIGRTMIN+13
    48) SIGRTMIN+14	49) SIGRTMIN+15	50) SIGRTMAX-14	51) SIGRTMAX-13	52) SIGRTMAX-12
    53) SIGRTMAX-11	54) SIGRTMAX-10	55) SIGRTMAX-9	56) SIGRTMAX-8	57) SIGRTMAX-7
    58) SIGRTMAX-6	59) SIGRTMAX-5	60) SIGRTMAX-4	61) SIGRTMAX-3	62) SIGRTMAX-2
    63) SIGRTMAX-1	64) SIGRTMAX

## pgrep/pkill

`pgrep` looks through the currently running processes and lists the process IDs which match the selection criteria to stdout.

    # pgrep sshd
    857
    22574
    22591
    27262
    27277


`pkill` will send the specified signal (by default SIGTERM) to each process  instead  of listing them on stdout

## pidof - find the process ID of a running program

`pidof` finds the process id's (pids) of the named programs. It prints those id's on the standard output.

`pidof` is similar to `pgrep` but without kill and regex capability.  

## Listing Load Average

You can use `top`

    # top -b -n 1 | grep "load average"
    top - 21:36:13 up  1:06,  3 users,  load average: 0.68, 0.96, 1.07

Or `w`

    # w | grep "load average"
    21:36:13 up  1:06,  3 users,  load average: 0.68, 0.96, 1.07

Or `uptime`

    uptime | grep "load average"
    21:36:13 up  1:06,  3 users,  load average: 0.68, 0.96, 1.07

---
[⬅️ Back](3-Operate-running-systems.md)
