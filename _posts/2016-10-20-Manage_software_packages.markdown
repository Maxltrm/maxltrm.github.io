---
layout: post
title:  "RHCSA 7 - Manage software packages"
date:   2016-10-20 18:30:54 +0200
categories: database how-to
---

### **Subscription Manager**

The subscription manager tools allows to:

* Register a system and associate it with a RedHat account, this allow to inventorize the system on the redhat customer portal.

* Subscribe a system to allow software upgrades and downloads.

* Enable repositories for downloading software packages, some repositories are enabled by default, others can be activated manually, such as updates and sources repositories.

* Keep track and review information about subscriptions from the system itself or from the customer portal.

* A system can be registered by gui by launching the subscription-manager-gui tool and selecting Applications> Other> Red Hat Subscription Manager or from the command line via the subscription-manager tool.

* Attach a system to the subscription that best matches the system being registered.

#### Register a system

```bash
subscription-manager register --username=yourusername --password=yourpassword
```

#### Show available subscriptions

```bash
subscription-manager list --available | less
```

#### Run the auto attach:

```bash
subscription-manager attach --auto
```

#### Show used subscritions

```bash
subscription-manager list --consumed
```

#### Unregister a system

```bash
subscription-manager unregister
```

#### Entitlement

An entitlement is a subscription connected to a system.

Once a system is registered, the certificates of the connected subscriptions are placed in the path **/etc/pki**.

Under /etc/pki there are some directories:

| PATH | Content description |
|:--------|--------:|
| /etc/pki/product | Certificates indicating the Red Hat products installed on the system. |
| /etc/pki/consumer | Certificates indicating the Red Hat account with which the system is registered. |
| /etc/pki/entitlement | Certificates indicating which subscriptions are connected to the system. |
|----

### **RPM Package Manager**
##### Originally Yellowdog Updater

It has its own database of installed packages.

#### **Usefull query**

#### Shows which package provided the indicated executable

```bash
rpm -qf /usr/bin/vi
vim-minimal-7.4.160-1.el7_3.1.x86_64
```

#### Queries all packages that require the functionality of a package

```bash
rpm -q --whatrequires <pkg_name>
```

#### Shows which files were installed with the package

```bash
rpm -ql vim-minimal
/etc/virc
/usr/bin/ex
/usr/bin/rvi
/usr/bin/rview
/usr/bin/vi
/usr/bin/view
/usr/share/man/man1/ex.1.gz
/usr/share/man/man1/rvi.1.gz
/usr/share/man/man1/rview.1.gz
/usr/share/man/man1/vi.1.gz
/usr/share/man/man1/view.1.gz
/usr/share/man/man1/vim.1.gz
/usr/share/man/man5/virc.5.gz
```

#### Shows the configuration files of the installed package

```bash
rpm -qc vim-minimal
/etc/virc
```

#### Shows the DOC files installed with the installed pakage

```bash
rpm -qd vim-minimal
/usr/share/man/man1/ex.1.gz
/usr/share/man/man1/rvi.1.gz
/usr/share/man/man1/rview.1.gz
/usr/share/man/man1/vi.1.gz
/usr/share/man/man1/view.1.gz
/usr/share/man/man1/vim.1.gz
/usr/share/man/man5/virc.5.gz
```

#### Query all

```bash
rpm   -qa

rpm -qa |grep vim
vim-enhanced-7.4.160-1.el7_3.1.x86_64
vim-minimal-7.4.160-1.el7_3.1.x86_64
vim-common-7.4.160-1.el7_3.1.x86_64
vim-filesystem-7.4.160-1.el7_3.1.x86_64
```

#### Show installation and uninstallation package scripts

```bash
rpm -q --scripts httpd
preinstall scriptlet (using /bin/sh):
# Add the "apache" group and user
/usr/sbin/groupadd -g 48 -r apache 2> /dev/null || :
/usr/sbin/useradd -c "Apache" -u 48 -g 48 \
-s /sbin/nologin -r -d /usr/share/httpd apache 2> /dev/null || :
postinstall scriptlet (using /bin/sh):

if [ $1 -eq 1 ] ; then
# Initial installation
systemctl preset httpd.service htcacheclean.service >/dev/null 2>&1 || :
fi
preuninstall scriptlet (using /bin/sh):

if [ $1 -eq 0 ] ; then
# Package removal, not upgrade
systemctl --no-reload disable httpd.service htcacheclean.service > /dev/null 2>&1 || :
systemctl stop httpd.service htcacheclean.service > /dev/null 2>&1 || :
fi
postuninstall scriptlet (using /bin/sh):

systemctl daemon-reload >/dev/null 2>&1 || :

# Trigger for conversion from SysV, per guidelines at:
# https://fedoraproject.org/wiki/Packaging:ScriptletSnippets#Systemd
posttrans scriptlet (using /bin/sh):
test -f /etc/sysconfig/httpd-disable-posttrans || \
/bin/systemctl try-restart httpd.service htcacheclean.service >/dev/null 2>&1 || :
rpm -qpl ftp://mirror.duomenucentras.lt/centos/7.5.1804/os/x86_64/Packages/chrony-3.2-2.el7.x86_64.rpm
/etc/NetworkManager/dispatcher.d/20-chrony
/etc/chrony.conf
/etc/chrony.keys
/etc/dhcp/dhclient.d/chrony.sh
/etc/logrotate.d/chrony
/etc/sysconfig/chronyd
/usr/bin/chronyc
/usr/lib/systemd/ntp-units.d/50-chronyd.list
/usr/lib/systemd/system/chrony-dnssrv@.service
/usr/lib/systemd/system/chrony-dnssrv@.timer
/usr/lib/systemd/system/chrony-wait.service
/usr/lib/systemd/system/chronyd.service
/usr/libexec/chrony-helper
/usr/sbin/chronyd
/usr/share/doc/chrony-3.2
/usr/share/doc/chrony-3.2/COPYING
/usr/share/doc/chrony-3.2/FAQ
/usr/share/doc/chrony-3.2/NEWS
/usr/share/doc/chrony-3.2/README
/usr/share/man/man1/chronyc.1.gz
/usr/share/man/man5/chrony.conf.5.gz
/usr/share/man/man8/chronyd.8.gz
/var/lib/chrony
/var/lib/chrony/drift
/var/lib/chrony/rtc
/var/log/chrony
rpm -qp --scripts ftp://mirror.duomenucentras.lt/centos/7.5.1804/os/x86_64/Packages/chrony-3.2-2.el7.x86_64.rpm
preinstall scriptlet (using /bin/sh):
getent group chrony > /dev/null || /usr/sbin/groupadd -r chrony
getent passwd chrony > /dev/null || /usr/sbin/useradd -r -g chrony \
-d /var/lib/chrony -s /sbin/nologin chrony
:
postinstall scriptlet (using /bin/sh):

if [ $1 -eq 1 ] ; then
# Initial installation
systemctl preset chronyd.service chrony-wait.service >/dev/null 2>&1 || :
fi
preuninstall scriptlet (using /bin/sh):

if [ $1 -eq 0 ] ; then
# Package removal, not upgrade
systemctl --no-reload disable chronyd.service chrony-wait.service > /dev/null 2>&1 || :
systemctl stop chronyd.service chrony-wait.service > /dev/null 2>&1 || :
fi
postuninstall scriptlet (using /bin/sh):

systemctl daemon-reload >/dev/null 2>&1 || :
if [ $1 -ge 1 ] ; then
# Package upgrade, not uninstall
systemctl try-restart chronyd.service >/dev/null 2>&1 || :
fi
```

### **Manage updates with yum**
##### Originally Yellowdog Updater

yum is a tool that can be used to install, update, remove and search for software packages.

yum is a meta package manager, by connecting to repositories it download some indexes from which to determine the dependencies for installing packages.

#### yum log path

```bash
/var/log/yum.log
```

#### **Common options**

#### Show installed and available packages

```bash
yum list httpd
Installed Packages
httpd.x86_64 2.4.6-45.el7.centos.4 @updates
Available Packages
httpd.x86_64 2.4.6-80.el7.centos base
```

#### Search for packages by keyword present only in the name and in the summary

```bash
yum search keyword
```

#### Search for packages that have 'web server' in the name, summary and description field

```bash
yum search all ‘web server’
```

#### Provides detailed info about the httpd package, including the disk space required for installation

```bash
yum info httpd

Installed Packages
Name : httpd
Arch : x86_64
Version : 2.4.6
Release : 45.el7.centos.4
Size : 9.4 M
Repo : installed
From repo : updates
Summary : Apache HTTP Server
URL : http://httpd.apache.org/
License : ASL 2.0
Description : The Apache HTTP Server is a powerful, efficient, and extensible
: web server.

Available Packages
Name : httpd
Arch : x86_64
Version : 2.4.6
Release : 80.el7.centos
Size : 2.7 M
Repo : base/7/x86_64
Summary : Apache HTTP Server
URL : http://httpd.apache.org/
License : ASL 2.0
Description : The Apache HTTP Server is a powerful, efficient, and extensible
: web server.
```

#### Lists the packages that provide the specified feature or file

```bash
yum provides pathname
yum provides httpd
yum provides /var/www/html
```

#### Show the files included in a package with repoquery

```bash
repoquery -ql httpd
```

#### Specify the repository

```bash
repoquery -repoid=testlocal -ql httpd
```

#### Install the indicated package including dependencies

```bash
yum install packagename
yum install httpd
```

#### Get and install a new version of the indicated package, including dependencies

```bash
yum update httpd
```

Generally the configuration files are kept, but they can be renamed if they are considered incompatible with the updated version.

#### Installs all relevant updates, or performs a system upgrade

```bash
yum update
```

#### Update kernel

Since a new kernel can only be tested by booting from it, its package is designed so that multiple versions can be installed.

If it fails to boot from the new kernel, the old version is still available.

#### lists installed and available versions

```bash
yum list kernel
```

#### Only updates the kernel and dependencies

```bash
yum update kernel
```

#### Update a system excluding specified package

```bash
yum update --exclude=kernel*
```

To make the exclusion permanent, edit the file /etc/yum.conf and add the string:

```bash
exclude=kernel* redhat-release*
```

#### Removes the indicated package and all the packages that have it as a dependency

```bash
yum remove package
```

#### Yum Groups

The groups are collections of software installed together for a particular purpose. In RHEL7 there are two types of groups.

Regular groups are collections of packages

Environment groups are collections of other groups.

The packages or groups provided by a group can be mandatory, default or optional (not installed unless specifically requested)

#### Shows installed and available groups

```bash
yum group list
yum grouplist
```

Some groups are normally installed with environment groups, and are not shown.

```bash
yum group list hidden
```

#### Show group info and a list divided by mandatory, default, optional

```bash
yum group info “Development Tools”
yum groupinfo “Development Tools”
```

Some groups may have a symbol in front

| Symbol | Description |
|:--------|--------:|
| = | The package is installed, and was installed as part of the group |
| + |  The package is not installed, it will be if the group is installed or updated. |
| - | The package is not installed, and it will not be if the group is installed or updated. |
| nothing | The package is installed, but was not installed as part of the group |
|----

#### Install the group and the related groups or packages mandatory and default

```bash
yum group install “Basic Web Server”
yum groupinstall “Basic Web Server”
```

#### Show transaction summary

```bash
yum history

ID | Command line | Date and time | Action(s) | Altered
-------------------------------------------------------------------------------
114 | install createrepo | 2018-06-08 17:05 | Install | 3
113 | install mlocate | 2018-06-01 14:56 | Install | 1
112 | -y install php55w-xml ph | 2018-03-02 11:46 | Install | 12
111 | -y install php55w php55w | 2018-03-02 11:45 | Install | 4
110 | remove php-common | 2018-03-02 11:45 | Erase | 53 E<
109 | update php | 2018-03-02 11:11 | Update | 19 >
```

#### Show detailed info about a transaction

```bash
yum history info 109
```

#### Undo a transaction

```bash
yum history undo 109
```

### Repository

#### view all available repositories

```bash
yum repolist all
```

#### enable the repository and update the file /etc/yum.repos.d/redhat.repo

```bash
yum-config-manager --enable rhel-7-server-extras-rpms
```

#### Create local repository

1. Save packages under a local path (es. /repolocal)
2. createrepo /repolocal                (Create repomd (xml-rpm-metadata) repository)
3. Create the configuration file under /etc/yum.repos.d

```bash
vi repolocal.repo

[repolocal]
name=repolocal
baseurl=file:///repolocal
enabled=1
gpgcheck=0
```

#### Third-party repositories

```bash
yum-config-manager --add-repo=“http://dl.fedoraproject.org/pub/epel/7/x86_64/”
```

Some repositories provide a configuration file (.repo) and the GPG public key as part of a package that can be downloaded and installed using yum localinstall.

An example of this is the EPEL volunteer project Extras Packages for Enterprise Linux, which provides software not supported by Redhat but compatible with Red Hat Enterprise Linux.

```bash
[EPEL]
name=EPEL Fedora 7
baseurl=http://dl.fedoraproject.org/pub/epel/7/x86_64/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
```

#### Import the key under /etc/pki/rpm-gpg/

```bash
rpm --import  http://dl.fedoraproject.org/pub/epel/RPM-GPG-KEY-EPEL-7
```

Install the GPG key RPM before installing signed packages, so that it can be verified that the package belongs to the installed key.

Otherwise yum will give a missing key error message (the --nogpgcheck option can be used to ignore missing keys).

```bash
yum install http://dl.fedoraproject.org/pub/epel/7/x86_64/Packages/e/epel-release-7-11.noarch.rpm
```

```bash
yum localinstall  curl-7.29.0-46.el7.x86_64.rpm            (installa il package scaricato in locale senza fare riferimento ai repository; ma si rivolge ai repository per le dipendenze).
```

Often the configuration file contains references to different repositories; any reference to a repository begins with a name (single word) in square brackets.

#### Disable/Enable a repository

```bash
yum-config-manager --enable|disable <nomerepository> 
```

Or temporarily with the yum --enablerepo command

```bash
yum search gpg --enablerepo=”EPEL Fedora 7″
```
