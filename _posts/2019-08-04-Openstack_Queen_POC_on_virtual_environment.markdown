---
layout: post
title:  "Openstack Queens POC on virtual environment with VirtualBMC - Part 1"
date:   2019-08-04 20:09:54 +0200
categories: Openstack how-to
---

With the ending (maybe for now, maybe forever) of my experience with Openstack I want to write this memorandum to establish and mantain some concepts about Openstack and the installation with triple-O, but since I haven't a private datacenter (for now), I'll simulate a physical environment with the help of VirtualBMC.

Since the procedure of installation it's quite long I'll split it in some parts.

### Part 1 - Install the Undercloud

#### Network Isolation diagram

(still in draft)

![](/assets/openstack-poc.png)

#### Virtual machines

3 x Controller 4Gb 4 vCPUs

3 x Compute 4Gb 4 vCPUs

1 x Undercloud 8Gb 4 vCPUs

#### [KVM-Host] Install VirtualBMC on a Python virtualenv

```
kvm-host ~ # virtualenv -p /usr/bin/python2.7 ooo-poc-venv

kvm-host ~ # source ooo-poc-venv/bin/activate

(ooo-poc-venv) kvm-host ~ # pip install virtualbmc
```

#### [KVM-Host] Configure vmbc

```
(ooo-poc-venv) kvm-host ~ # vbmc add Controller-0 --port 6230 --username admin --password ooo-poc
(ooo-poc-venv) kvm-host ~ # vbmc add Controller-1 --port 6231 --username admin --password ooo-poc
(ooo-poc-venv) kvm-host ~ # vbmc add Controller-2 --port 6232 --username admin --password ooo-poc

(ooo-poc-venv) kvm-host ~ # vbmc add Compute-0 --port 6233 --username admin --password ooo-poc
(ooo-poc-venv) kvm-host ~ # vbmc add Compute-1 --port 6234 --username admin --password ooo-poc
(ooo-poc-venv) kvm-host ~ # vbmc add Compute-2 --port 6235 --username admin --password ooo-poc

(ooo-poc-venv) kvm-host ~ # vbmc start Controller-0
(ooo-poc-venv) kvm-host ~ # vbmc start Controller-1
(ooo-poc-venv) kvm-host ~ # vbmc start Controller-2

(ooo-poc-venv) kvm-host ~ # vbmc start Compute-0
(ooo-poc-venv) kvm-host ~ # vbmc start Compute-1
(ooo-poc-venv) kvm-host ~ # vbmc start Compute-2

(ooo-poc-venv) kvm-host ~ # vbmc list
+--------------+---------+---------+------+
| Domain name  | Status  | Address | Port |
+--------------+---------+---------+------+
| Compute-0    | running | ::      | 6233 |
| Compute-1    | running | ::      | 6234 |
| Compute-2    | running | ::      | 6235 |
| Controller-0 | running | ::      | 6230 |
| Controller-1 | running | ::      | 6231 |
| Controller-2 | running | ::      | 6232 |
+--------------+---------+---------+------+

```

#### [KVM-Host] Test virtualbmc

```
(ooo-poc-venv) kvm-host ~ #   ipmitool -I lanplus -U admin -P ooo-poc -H 192.168.122.1 -p 6230 power status
Chassis Power is on
(ooo-poc-venv) kvm-host ~ #   ipmitool -I lanplus -U admin -P ooo-poc -H 192.168.122.1 -p 6231 power status
Chassis Power is on
(ooo-poc-venv) kvm-host ~ #   ipmitool -I lanplus -U admin -P ooo-poc -H 192.168.122.1 -p 6233 power status
Chassis Power is on
```

#### [Director] Configure director

* Update the system and create stack user

```
# yum update -y
# useradd stack
# passwd stack
# echo "stack ALL=(root) NOPASSWD:ALL" >> /etc/sudoers.d/stack
# chmod 0440 /etc/sudoers.d/stack

# su - stack

[stack@director ~]$ sudo hostnamectl set-hostname director.lab.lan
[stack@director ~]$ sudo hostnamectl set-hostname --transient director.lab.lan
```

* Configure hosts file

An entry for the systems FQDN hostname is also needed in /etc/hosts.

```
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4 director director.lab.lan
```

* Install python2-tripleo-repos

```
[stack@director ~]$ sudo yum install -y https://trunk.rdoproject.org/centos7/current/python2-tripleo-repos-0.0.1-0.20190724014728.1cf6e0b.el7.noarch.rpm
```

* Enable queens repositories

```
[stack@director ~]$ sudo tripleo-repos -b queens current
```

* Install tripleo client

```
[stack@director ~]$ sudo yum install -y python-tripleoclient
```

#### Install the undercloud

1. Compile undercloud configuration file

```
[stack@director ~]$ cp /usr/share/instack-undercloud/undercloud.conf.sample ~/undercloud.conf
```

This is my configuration file

```
[stack@director ~]$ cat undercloud.conf | egrep -v "^$|^#"

[DEFAULT]
undercloud_hostname = director.lab.lan
local_ip = 192.168.24.2/24
undercloud_public_host = 192.168.24.3
undercloud_admin_host = 192.168.24.4
undercloud_service_certificate =
generate_service_certificate = true
local_interface = eth1
local_mtu = 1500
scheduler_max_attempts = 6
ipxe_enabled = true
masquerade_network = 192.168.24.0/24
[auth]
[ctlplane-subnet]
cidr = 192.168.24.0/24
dhcp_start = 192.168.24.5
dhcp_end = 192.168.24.24
inspection_iprange = 192.168.24.100,192.168.24.120
```

2. Run the installation

```
stack@director ~]$ openstack undercloud install
2019-08-03 11:50:35,226 INFO: Logging to /home/stack/.instack/install-undercloud.log

[...]
[...]
[...]

2019-08-03 12:10:50,360 INFO:
#############################################################################
Undercloud install complete.

The file containing this installation's passwords is at
/home/stack/undercloud-passwords.conf.

There is also a stackrc file at /home/stack/stackrc.

These files are needed to interact with the OpenStack services, and should be
secured.

#############################################################################
```

#### Build overcloud images

```
$ source stackrc

(undercloud) [stack@director ~]$ export DIB_YUM_REPO_CONF="/etc/yum.repos.d/delorean*"

(undercloud) [stack@director ~]$ openstack overcloud image build

[...]
[...]
[...]

2019-08-03 11:22:48.350 | Build completed successfully
```

#### Upload the images in the undercloud glance

```
(undercloud) [stack@director ~]$ openstack overcloud image upload

(undercloud) [stack@director ~]$ openstack image list
+--------------------------------------+------------------------+--------+
| ID                                   | Name                   | Status |
+--------------------------------------+------------------------+--------+
| 357a9921-5137-4a9c-9707-9dc45660927d | bm-deploy-kernel       | active |
| f256ea6a-23ad-4d85-b506-e87e39245075 | bm-deploy-ramdisk      | active |
| 4f2673ef-eadd-4c08-8a94-45cd2bdb4d87 | overcloud-full         | active |
| 93d5e21f-cb23-40b7-b575-d06f22510e70 | overcloud-full-initrd  | active |
| 9e2ccfbf-fbfa-44e5-bcce-535a2c773653 | overcloud-full-vmlinuz | active |
+--------------------------------------+------------------------+--------+
```

Next step: Import and introspect baremetal node
