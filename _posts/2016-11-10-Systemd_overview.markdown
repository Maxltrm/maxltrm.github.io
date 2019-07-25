---
layout: post
title:  "RHCSA 7 - Systemd Overview"
date:   2016-11-10 13:09:54 +0200
categories: linux notes
---

#### Systemd allows:

* The parallel start of services at boot time
* To start demons on-demand
* To track processes with cgroups
* To make system status snapshots and to restore the system from said snapshots
* To configure mountpoints as systemd targets
* Automatic management of service dependencies, which can avoid long timeouts, for example by not activating a network service if the network is not available.

Systemd is the first process started at boot time, and the last one to stop before the machine shuts down.
It allows the machine to boot faster.

#### Systemd manages:

* Services (name.service)
* Targets (name.target)
* Devices (name.device)
* Filesystem mountpoints (name.mount)
* Sockets (name.socket)
* Tmpfs at boot time (systemctl enable tmp.mount)

#### Boot process with systemd

1. The BIOS executes POST
2. The BIOS reads the MBR and starts GRUB
3. The Boot Loader loads the vmlinux kernel in memory and it extracts the contents of the initramfs into the tmpfs.
4. The kernel loads the driver modules needed to access the root filesystem from the initramfs.
5. The kernel loads systemd with PID 1, systemd is the father process of all processes.
1. Systemd reads the configuration files from the directory /etc/systemd/system.
2. Systemd reads the link **/etc/systemd/system/default.target**, for example **/usr/lib/systemd/system/multi-user.target**, in order to determine the system target then determine the services that will be started by systemd.

#### System targets

System targets represent what used to be called a **runlevel** with **systemV**

The default system-state target can be define linking the following file:

```
/etc/systemd/system/default.target
```

to one of the targets listed in:

```
/usr/lib/systemd/system/multi-user.target
```

or directly with systemctl:

```
systemctl set-default multi-user.target
```

#### Targets and related runlevels

* graphical.taget -> runlevel5.target
* multi-user.target -> runlevel2.target
* multi-user.target -> runlevel3.target
* runlevel4.target -> multi-user.target
* poweroff.target -> runlevel0.target
* reboot.target -> runlevel6.target
* rescure.target -> runleve1.target

runlevel*.target records are symlinks.

#### Managing targets

View default targets:

```
# systemctl get-default
```

View active targets on the system:

```
# systemctl list-units --type target
```

Change default target:

```
# systemct set-default <target>
```

Change the currently active target:

```
# systemctl isolate <target>
```

#### Shutting down - suspending - rebooting

Halt the system

```
# systemctl halt
```

Hibernate the system

```
# systemctl hibernate
```

Hibernate and suspend

```
# systemct hybrid-sleep
```

Power off the system

```
# systemctl poweroff
```

Reboot the system

```
# systemctl reboot
```

Suspend the system

```
# systemctl suspend
```

#### Start - Stop - Enable - Disable a service

```
# systemctl start|stop|enable|disable sshd
```

Check if a service is enabled

```
# systemctl is-enabled sshd
```

Check if a service is currently active

```
# systemctl is-active sshd
```

Show a detailed summary of the status of a service, including cgroup details

```
# systemctl status sshd -l
```

Sometimes, services are grouped in a cgroup in order to better manage their access to the available resources.

For example, the httpd cgroup is httpd.service, which represents a System Slice.
System slices separate groups in different categories.
**systemd-cgls** shows the hierarchy of slices e cgroups.
