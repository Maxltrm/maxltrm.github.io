---
layout: post
title:  "Linux process management"
date:   2018-07-25 16:09:54 +0200
categories: linux notes
---

Linux is a multiprocessing operating system with **preemptable processes**, it means that the kernel can interrupt the execution of a process, arbitrarily, also in kernel space, and can replace the CPU with another process without the process being informed.

The operating system tracks how long each process holds the CPU and periodically activates the scheduler.

A CPU can run in differend execution states.

All starndard Unix Kernel use only two execution states: **User Mode** and **Kernel Mode**

Unix-like Kernel implement programming constructs called **system calls** to execute hardware-dependent CPU instructions to switch from User Mode to Kernel Mode.

A program usually executes in **User Mode**, whenever a process makes a system call, the hardware changes the privilege mode from User Mode to Kernel Mode, and the process starts the execution of a kernel procedure with a strictly limited purpose, in this way, the operating system acts within the execution context of the process in order to satisfy its request, whenever the request is fully satisfied, the kernel procedure forces the hardware to return to User Mode and the process continues its execution from the instruction following the system call.

Unix makes a neat distinction between the process and the program it is executing. To that end, the fork( ) and _exit( ) system calls are used respectively to create a new process and to terminate it, while an exec( )-like system call is invoked to load a new program. After such a system call is executed, the process resumes execution with a brand new address space containing the loaded program.

The process that invokes a fork( ) is the parent, while the new process is its child. Parents and children can find one another because the data structure describing each process includes a pointer to its immediate parent and pointers to all its immediate children.

Each process is represented by a process descriptor, a [task_struct](https://github.com/torvalds/linux/blob/master/include/linux/sched.h#L637) type that contains information about current process state (the process's priority, whether it is running on a CPU or blocked on an event, what address space has been assigned to it, which files it is allowed to address, and so on), when the kernel decides to resume executing a process, it uses the proper process descriptor fields to load the CPU registers. Because the stored value of the program counter points to the instruction following the last instruction executed, the process resumes execution at the point where it was stopped.

The following are the possible process states:

| PS State | State | Description |
|:--------|:-------:|--------:|
| R | TASK_RUNNING | The process is either executing on a CPU or waiting to be executed. |
| S | TASK_INTERRUPTIBLE | The process is suspended (sleeping) until some condition becomes true. Raising a hardware interrupt, releasing a system resource the process is waiting for, or delivering a signal are examples of conditions that might wake up the process (put its state back to TASK_RUNNING). |
|  | TASK_UNINTERRUPTIBLE | Like TASK_INTERRUPTIBLE, except that delivering a signal to the sleeping process leaves its state unchanged. This process state is seldom used. It is valuable, however, under certain specific conditions in which a process must wait until a given event occurs without being interrupted. For instance, this state may be used when a process opens a device file and the corresponding device driver starts probing for a corresponding hardware device. The device driver must not be interrupted until the probing is complete, or the hardware device could be left in an unpredictable state. |
| T | TASK_STOPPED | Process execution has been stopped; the process enters this state after receiving a SIGSTOP, SIGTSTP, SIGTTIN, or SIGTTOU signal. |
|  | TASK_TRACED | Process execution has been stopped by a debugger. When a process is being monitored by another (such as when a debugger executes a ptrace( ) system call to monitor a test program), each signal may put the process in the TASK_TRACED state. |
| Z | EXIT_ZOMBIE | Process execution is terminated, but the parent process has not yet issued a wait4( ) or waitpid( ) system call to return information about the dead process.[*] Before the wait( )-like call is issued, the kernel cannot discard the data contained in the dead process descriptor because the parent might need it. (See the section "Process Removal" near the end of this chapter.) |
| X | EXIT_DEAD | The final state: the process is being removed by the system because the parent process has just issued a wait4( ) or waitpid( ) system call for it. Changing its state from EXIT_ZOMBIE to EXIT_DEAD avoids race conditions due to other threads of execution that execute wait( )-like calls on the same process (see Chapter 5). |
|----

It's possible to see process state with ps:

```
ps -ef -o pid,comm,state
```

#### ps command common options

```
-e	show all processes
-f	full-format
-F	extra full-format
-L	show also threads
-u	show user-oriented format
-o	user-defined format
```

#### Show processes with BSD syntax

```
# ps axu
```

#### Specify a user-defined output format

```
# ps -axo pid,comm,prio,nice
```

#### Sort the output

```
# ps aux --sort=pcpu
```

### Signals

A signal is a software interrupt sended to a process

#### Signal default action

Term	Cause the stop (exit) of the program  (SIGTERM, SIGKILL)
Core	Save a memory image (core dump) and then exit
Stop	Suspende (pause) a program and wait for continue (resume)  (SIGSTOP, SIGCONT)

#### Keyboard Signals for foreground process

CTRL-z 	Supend the process (bg or fg)
CTRL-c	Stop
CTRL-\ 	Generate a core dump

[Posix signals List](https://en.wikipedia.org/wiki/Signal_(IPC)

### Send signal to process

#### kill

The kill command can be used to send any kind of segnals, not only to terminate a process

```
# kill 8590      (invia un segnale SIGTERM al processo)
```

```
# kill -SIGKILL 8590   (invia un segnale SIGKILL al processo)
```

##### List signal names

```
# kill -l
```

#### killall

killall can be used to send sinagl to a group of processes with a criteria using: command name, process of a specific user, system-wide processes

```
killall -signal command_pattern
```

##### Send a SIGTERM to all "httpd" named process
```
# killall httpd
```

##### Send a SIGTERM to all process owned by the user postfix

```
# killall -u postfix
```

##### Send a SIGKILL to all process owned by the user apache

```
# killall -9 -u apache
```

#### pkill

Uses more advanzed selection criteria

```
pkill command_pattern
pkill -signal command_pattern
pkill -G GID command_pattern
```

Only match processes whose parent process ID is listed.

```
pkill -P PPID command_pattern
```

#### Send a SIGTERM to process residenti in a tty terminal

```
pkill -t terminal_name -u UID command_pattern
```

#### w command

```
# w
11:54:28 up 20:07, 2 users, load average: 0.00, 0.01, 0.05
USER  TTY   FROM            LOGIN@   IDLE   JCPU   PCPU   WHAT
root    tty1                         11:51             3:00   0.00s      0.00s    -bash
root   pts/0 192.168.56.1 Mon15          4.00s  0.56s     0.01s      w
```

pts/n   da ambiente grafico  (pseudo-terminal)
ttyn    system-console

TTY + FROM   per determinare user location
JCPU  risorse cpu della sessione compresi processi in background
PCPU  risorse cpu processi in foreground

```
pgrep  -l  -u  ldapuser01      ( -l, --list-name     List the process name as well as the process ID)
2978 bash
2998 sleep
3000 sleep
```

```
w -u ldapuser01
13:13:40 up 21:27, 3 users, load average: 0.00, 0.01, 0.05
USER TTY FROM LOGIN@ IDLE JCPU PCPU WHAT
ldapuser pts/1 192.168.56.1 12:41 20.00s 0.04s 0.04s -bash
```
pkill -t  pts/1        (mantiene la bash login shell)

pkill -SIGTERM -t  pts/1        ( -t  terminal.  Mantiene il processo bash, session leader in questo caso)

pkill -SIGTERM -SIGKILL -t  pts/1   (Ammazza anche il processo bash)

pstree -p ldapuser01        ( -p Show PIDs )

bash(3521)---pstree(3544)
--sleep(3541)
--sleep(3542)
--sleep(3543)

pkill -P   3521          (-P Only match processes whose parent process ID is listed.  Mantiene il Parent process 3521)

#### Load average

Provided by commands top, uptime, w
The three values indicate the load of last 1, 5 and 15 minutes
Divide the value by the number of logical CPUs. A number less than 1 indicates a satisfactory use of resources and a minimum waiting time; a value greater than 1 indicates a saturation of resources and prolonged waiting times (disk I / O and network).

```
# uptime
18:20:53 up 184 days, 4:08, 1 user, load average: 0.00,  0.01,  0.05
```

```
# top
top - 19:45:00 up 184 days, 5:32, 1 user, load average: 0.00, 0.01, 0.08

Tasks: 121 total, 1 running, 119 sleeping, 1 stopped, 0 zombie
%Cpu(s): 0.2 us, 0.0 sy, 0.0 ni, 99.8 id, 0.0 wa, 0.0 hi, 0.0 si, 0.0 st
KiB Mem : 1782272 total, 1446944 free, 222028 used, 113300 buff/cache
KiB Swap: 3801084 total, 3479088 free, 321996 used. 1408040 avail Mem

PID USER PR NI VIRT RES SHR S %CPU %MEM TIME+ COMMAND
2378 root 20  0 357084 3824 1500 S 0.7 0.2 296:34.43 docker-containe
425 root 20    0 0 0 0 S 0.3 0.0 56:40.84 xfsaild/dm-0
```

VIRT  Virtual memory, all the memory used by the process, including resident set, shared libraries, and each mapped or swapped memory page (VSZ of the ps command)
RES   Resident memory, the physical memory used by the process, including each resident shared object (RSS of the ps command)
TIME  total time since the process started; could include the cumulative time of previous children processes.

#### Nice e renice

The nice levels can be seen in various ways  (top, ps, gnome-system-monitor).
With the top command, the NI columns (nice, between -20 and 19) and PR (priority, between 0 and 39) show values related to the nice.

```
Tasks: 120 total, 1 running, 119 sleeping, 0 stopped, 0 zombie
%Cpu(s): 0.1 us, 0.0 sy, 0.0 ni, 99.9 id, 0.0 wa, 0.0 hi, 0.0 si, 0.0 st
KiB Mem : 1782272 total, 86180 free, 259888 used, 1436204 buff/cache
KiB Swap: 3801084 total, 3500996 free, 300088 used. 1297444 avail Mem

PID USER PR NI VIRT RES SHR S %CPU %MEM TIME+ COMMAND
7303 root 20 0  157724 2164 1492 R 6.2 0.1 0:00.01 top
1    root 20 0  190756 2892 1896 S 0.0 0.2 16:48.10 systemd
2    root 20 0  0 0 0 S 0.0 0.0 0:02.77 kthreadd
3    root 20 0  0 0 0 S 0.0 0.0 0:04.29 ksoftirqd/0
5    root 0 -20 0 0 0 S 0.0 0.0 0:00.00 kworker/0:0H
7    root rt 0  0 0 0 S 0.0 0.0 0:01.03 migration/0
8    root 20 0  0 0 0 S 0.0 0.0 0:00.00 rcu_bh
9    root 20 0  0 0 0 S 0.0 0.0 28:32.15 rcu_sched
10   root rt 0  0 0 0 S 0.0 0.0 1:10.52 watchdog/0
11   root rt 0  0 0 0 S 0.0 0.0 1:10.99 watchdog/1
12   root rt 0  0 0 0 S 0.0 0.0 0:01.16 migration/1
```

ps axo pid,com,nice  --sort =nice      (ordina i processi per nice)

Processes that report a - like nice level, are running with a different scheduling policy, and are almost certainly considered higher priority by the scheduler.

The scheduler policy can be viewed by requesting the cls field in the ps command.

```
ps axo pid,com,nice,cls 
```

TS        processo gira con SCHED_NORMAL e usa nice levels

Tutto il resto utilizza una scheduler policy differente.

When a process is launched, it normally inherits the nice level from the parent process. A process launched by the command line then inherits the nice level of the shell from which it is launched, usually nice level 0.

To run a process specifying the nice level use the command nice:

```
nice <command>        Run the process with run level 10
nice -n 15 sleep 600  Run the process with rul level 15
```
It's possible to modify the nice level using the command renice:

```
renice -n -7 processo
```

Non privileged users can run process with a positive nice level, only root can run processes with negative nice level, from -20 to -1

Normal users can increase the nice level but only root can decrease it.

The top command can also be used to change the nice level, press r and then indicate PID and new nice level.

Links:

An interesting article from Brendan Gregg blog about load average [link](http://www.brendangregg.com/blog/2017-08-08/linux-load-averages.html){:target="_blank"}
