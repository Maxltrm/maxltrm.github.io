---
layout: post
title:  "Install OMD on Raspbian"
date:   2018-09-08 18:20:54 +0200
categories: linux notes
---

#### Install OMD

```
cd ~
wget -q "https://labs.consol.de/repo/stable/RPM-GPG-KEY" -O - | sudo apt-key add -
echo "deb http://labs.consol.de/repo/stable/debian $(lsb_release -cs) main" > /etc/apt/sources.list.d/labs-consol-stable.list
apt-get update
apt-get install omd-labs-edition
omd setup
We will install missing packages from your operating system and setup the
system apache daemon (add configuration files and modules needed by omd)
(yes/NO): yes
Going to execute 'apt-get -y update ; apt-get -y install time traceroute curl dialog dnsutils fping graphviz libapache2-mod-fcgid libboost-atomic1.62.0 libboost-chrono1.62.0 libboost-date-time1.62.0 libboost-program-options1.62.0 libboost-system1.62.0 libboost-regex1.62.0 libboost-thread1.62.0 libdbi1 libevent-2.0-5 libgd3 libicu57 libltdl7 libnet-snmp-perl libpango1.0-0 libperl5.24 libreadline7 libsnmp-perl libuuid1 libpython2.7 mariadb-server|mysql-server|virtual-mysql-server patch php php-cgi php-cli php-gd php-mbstring php-sqlite3 php-pear rsync snmp unzip xinetd python-ldap libradcli4'
```

#### Create omd site

```
omd create mon
Adding /omd/sites/mon/tmp to /etc/fstab.
Creating temporary filesystem /omd/sites/mon/tmp...OK
Restarting Apache...OK
Created new site mon with version 2.60-labs-edition.

The site can be started with omd start mon.
The default web UI is available at http://receiver/mon/
The admin user for the web applications is omdadmin with password omd.
Please do a su - mon for administration of this site.
```

#### Start OMD

```
OMD[mon]:~$ omd start
Starting rrdcached...OK
Starting npcd...OK
Starting naemon...OK
Starting dedicated Apache for site mon...OK
Initializing Crontab...OK
OMD[mon]:~$ omd status
rrdcached: running
npcd: running
naemon: running
apache: running
crontab: running
-----------------------
Overall state: running
```
