---
layout: post
title:  "Install freeIPA Server"
date:   2016-12-11 19:09:54 +0200
categories: linux notes
---

#### Prerequisites

Configure a private DNS or register a domain

My Environment:

| Hostname| IP | OS | Domain |
|:--------|:-------:|:-------:|--------:|
| dns01 |  10.10.10.14/28 | Centos 7.4 | lab.lan |
| ipa   |  10.10.10.13/28 | Centos 7.4 | lab.lan |
| nas01 |  10.10.10.12/28 | Centos 7.4 | lab.lan |
| web01 |  10.10.10.11/28 | Centos 7.0 | lab.lan |
| web02 |  10.10.10.10/28 | Centos 7.0 | lab.lan |
|----

#### Open firewall ports

```bash
[root@ipa ~]# firewall-cmd --permanent --add-port={80/tcp,443/tcp,389/tcp,636/tcp,88/tcp,464/tcp,53/tcp,88/udp,464/udp,53/udp,123/udp,749/tcp,749/udp}
[root@ipa ~]# firewall-cmd --reload
```

#### Verify FQDN resolution

```bash
yum install bind-utils
dig +short ipa.lab.lan A
10.10.10.13
dig +short -x 10.10.10.13
ipa.lab.lan.
```

#### Install random number generator

```bash
[root@ipa ~]# yum install rng-tools
[root@ipa ~]# systemctl start rngd; systemctl enable rngd
```

If the start does not occur, a hardware entropy generator may not be present, modify ExecStart in the service /etc/systemd/system/rngd.service as follows:

```bash
[root@ipa ~]# vim /etc/systemd/system/rngd.service
 ExecStart=/sbin/rngd -f -r /dev/urandom
```

#### Install IPA server package

```bash
[root@ipa ~]# yum install ipa-server
```

#### Run IPA server install procedure

```bash
[root@ipa ~]# ipa-server-install

The log file for this installation can be found in /var/log/ipaserver-install.log
==============================================================================
This program will set up the IPA Server.

This includes:
 * Configure a stand-alone CA (dogtag) for certificate management
 * Configure the Network Time Daemon (ntpd)
 * Create and configure an instance of Directory Server
 * Create and configure a Kerberos Key Distribution Center (KDC)
 * Configure Apache (httpd)
 * Configure the KDC to enable PKINIT

To accept the default shown in brackets, press the Enter key.

Do you want to configure integrated DNS (BIND)? [no]:

Enter the fully qualified domain name of the computer
on which you're setting up server software. Using the form
<hostname>.<domainname>
Example: master.example.com.


Server host name [ipa.lab.lan]:ipa.lab.lan

The domain name has been determined based on the host name.

Please confirm the domain name [lab.lan]: lab.lan

The kerberos protocol requires a Realm name to be defined.
This is typically the domain name converted to uppercase.

Please provide a realm name [LAB.LAN]:
Certain directory server operations require an administrative user.
This user is referred to as the Directory Manager and has full access
to the Directory for system management tasks and will be added to the
instance of directory server created for IPA.
The password must be at least 8 characters long.

Directory Manager password:
Password (confirm):

The IPA server requires an administrative user, named 'admin'.
This user is a regular system account used for IPA server administration.

IPA admin password:
Password (confirm):


The IPA Master Server will be configured with:
Hostname: ipa.lab.lan
IP address(es): 10.10.10.13
Domain name: ipa.lab.lan
Realm name: IPA.LAB.LAN

Continue to configure the system with these values? [no]: yes

The following operations may take some minutes to complete.
Please wait until the prompt is returned.

Configuring NTP daemon (ntpd)
 [1/4]: stopping ntpd
 [2/4]: writing configuration
 [3/4]: configuring ntpd to start on boot
 [4/4]: starting ntpd
Done configuring NTP daemon (ntpd).
Configuring directory server (dirsrv). Estimated time: 30 seconds
 [1/45]: creating directory server instance
 [2/45]: enabling ldapi
 [3/45]: configure autobind for root
 [4/45]: stopping directory server
 [5/45]: updating configuration in dse.ldif
 [6/45]: starting directory server
 [7/45]: adding default schema
 [8/45]: enabling memberof plugin
 [9/45]: enabling winsync plugin
 [10/45]: configuring replication version plugin
 [11/45]: enabling IPA enrollment plugin
 [12/45]: configuring uniqueness plugin
 [13/45]: configuring uuid plugin
 [14/45]: configuring modrdn plugin
 [15/45]: configuring DNS plugin
 [16/45]: enabling entryUSN plugin
 [17/45]: configuring lockout plugin
 [18/45]: configuring topology plugin
 [19/45]: creating indices
 [20/45]: enabling referential integrity plugin
 [21/45]: configuring certmap.conf
 [22/45]: configure new location for managed entries
 [23/45]: configure dirsrv ccache
 [24/45]: enabling SASL mapping fallback
 [25/45]: restarting directory server
 [26/45]: adding sasl mappings to the directory
 [27/45]: adding default layout
 [28/45]: adding delegation layout
 [29/45]: creating container for managed entries
 [30/45]: configuring user private groups
 [31/45]: configuring netgroups from hostgroups
 [32/45]: creating default Sudo bind user
 [33/45]: creating default Auto Member layout
 [34/45]: adding range check plugin
 [35/45]: creating default HBAC rule allow_all
 [36/45]: adding entries for topology management
 [37/45]: initializing group membership
 [38/45]: adding master entry
 [39/45]: initializing domain level
 [40/45]: configuring Posix uid/gid generation
 [41/45]: adding replication acis
 [42/45]: activating sidgen plugin
 [43/45]: activating extdom plugin
 [44/45]: tuning directory server
 [45/45]: configuring directory to start on boot
Done configuring directory server (dirsrv).
Configuring Kerberos KDC (krb5kdc)
 [1/10]: adding kerberos container to the directory
 [2/10]: configuring KDC
 [3/10]: initialize kerberos container
 [4/10]: adding default ACIs
 [5/10]: creating a keytab for the directory
 [6/10]: creating a keytab for the machine
 [7/10]: adding the password extension to the directory
 [8/10]: creating anonymous principal
 [9/10]: starting the KDC
 [10/10]: configuring KDC to start on boot
Done configuring Kerberos KDC (krb5kdc).
Configuring kadmin
 [1/2]: starting kadmin
 [2/2]: configuring kadmin to start on boot
Done configuring kadmin.
Configuring certificate server (pki-tomcatd). Estimated time: 3 minutes
 [1/29]: configuring certificate server instance
 [2/29]: exporting Dogtag certificate store pin
 [3/29]: stopping certificate server instance to update CS.cfg
 [4/29]: backing up CS.cfg
 [5/29]: disabling nonces
 [6/29]: set up CRL publishing
 [7/29]: enable PKIX certificate path discovery and validation
 [8/29]: starting certificate server instance
 [9/29]: configure certmonger for renewals
 [10/29]: requesting RA certificate from CA
 [11/29]: setting up signing cert profile
 [12/29]: setting audit signing renewal to 2 years
 [13/29]: restarting certificate server
 [14/29]: publishing the CA certificate
 [15/29]: adding RA agent as a trusted user
 [16/29]: authorizing RA to modify profiles
 [17/29]: authorizing RA to manage lightweight CAs
 [18/29]: Ensure lightweight CAs container exists
 [19/29]: configure certificate renewals
 [20/29]: configure Server-Cert certificate renewal
 [21/29]: Configure HTTP to proxy connections
 [22/29]: restarting certificate server
 [23/29]: updating IPA configuration
 [24/29]: enabling CA instance
 [25/29]: migrating certificate profiles to LDAP
 [26/29]: importing IPA certificate profiles
 [27/29]: adding default CA ACL
 [28/29]: adding 'ipa' CA entry
 [29/29]: configuring certmonger renewal for lightweight CAs
Done configuring certificate server (pki-tomcatd).
Configuring directory server (dirsrv)
 [1/3]: configuring TLS for DS instance
 [2/3]: adding CA certificate entry
 [3/3]: restarting directory server
Done configuring directory server (dirsrv).
Configuring ipa-otpd
 [1/2]: starting ipa-otpd
 [2/2]: configuring ipa-otpd to start on boot
Done configuring ipa-otpd.
Configuring ipa-custodia
 [1/5]: Generating ipa-custodia config file
 [2/5]: Making sure custodia container exists
 [3/5]: Generating ipa-custodia keys
 [4/5]: starting ipa-custodia
 [5/5]: configuring ipa-custodia to start on boot
Done configuring ipa-custodia.
Configuring the web interface (httpd)
 [1/22]: stopping httpd
 [2/22]: setting mod_nss port to 443
 [3/22]: setting mod_nss cipher suite
 [4/22]: setting mod_nss protocol list to TLSv1.0 - TLSv1.2
 [5/22]: setting mod_nss password file
 [6/22]: enabling mod_nss renegotiate
 [7/22]: disabling mod_nss OCSP
 [8/22]: adding URL rewriting rules
 [9/22]: configuring httpd
 [10/22]: setting up httpd keytab
 [11/22]: configuring Gssproxy
 [12/22]: setting up ssl
 [13/22]: configure certmonger for renewals
 [14/22]: importing CA certificates from LDAP
 [15/22]: publish CA cert
 [16/22]: clean up any existing httpd ccaches
 [17/22]: configuring SELinux for httpd
 [18/22]: create KDC proxy config
 [19/22]: enable KDC proxy
 [20/22]: starting httpd
 [21/22]: configuring httpd to start on boot
 [22/22]: enabling oddjobd
Done configuring the web interface (httpd).
Configuring Kerberos KDC (krb5kdc)
 [1/1]: installing X509 Certificate for PKINIT
Done configuring Kerberos KDC (krb5kdc).
Applying LDAP updates
Upgrading IPA:. Estimated time: 1 minute 30 seconds
 [1/9]: stopping directory server
 [2/9]: saving configuration
 [3/9]: disabling listeners
 [4/9]: enabling DS global lock
 [5/9]: starting directory server
 [6/9]: upgrading server
 [7/9]: stopping directory server
 [8/9]: restoring configuration
 [9/9]: starting directory server
Done.
Restarting the KDC
Please add records in this file to your DNS system: /tmp/ipa.system.records.jBoWLR.db
Configuring client side components
Using existing certificate '/etc/ipa/ca.crt'.
Client hostname: ipa.lab.lan
Realm: IPA.LAB.LAN
DNS Domain: ipa.lab.lan
IPA Server: ipa.lab.lan
BaseDN: dc=ipa,dc=lab,dc=lan

Skipping synchronizing time with NTP server.
New SSSD config will be created
Configured sudoers in /etc/nsswitch.conf
Configured /etc/sssd/sssd.conf
trying https://ipa.lab.lan/ipa/json
[try 1]: Forwarding 'schema' to json server 'https://ipa.lab.lan/ipa/json'
trying https://ipa.lab.lan/ipa/session/json
[try 1]: Forwarding 'ping' to json server 'https://ipa.lab.lan/ipa/session/json'
[try 1]: Forwarding 'ca_is_enabled' to json server 'https://ipa.lab.lan/ipa/session/json'
Systemwide CA database updated.
Adding SSH public key from /etc/ssh/ssh_host_rsa_key.pub
Adding SSH public key from /etc/ssh/ssh_host_ecdsa_key.pub
Adding SSH public key from /etc/ssh/ssh_host_ed25519_key.pub
[try 1]: Forwarding 'host_mod' to json server 'https://ipa.lab.lan/ipa/session/json'
Could not update DNS SSHFP records.
SSSD enabled
Configured /etc/openldap/ldap.conf
Configured /etc/ssh/ssh_config
Configured /etc/ssh/sshd_config
Configuring ipa.lab.lan as NIS domain.
Client configuration complete.
The ipa-client-install command was successful

==============================================================================
Setup complete

Next steps:
 1. You must make sure these network ports are open:
 TCP Ports:
 * 80, 443: HTTP/HTTPS
 * 389, 636: LDAP/LDAPS
 * 88, 464: kerberos
 UDP Ports:
 * 88, 464: kerberos
 * 123: ntp

2. You can now obtain a kerberos ticket using the command: 'kinit admin'
 This ticket will allow you to use the IPA tools (e.g., ipa user-add)
 and the web user interface.

Be sure to back up the CA certificates stored in /root/cacert.p12
These files are required to create replicas. The password for these
files is the Directory Manager password
```

```bash
[root@ipa ~]# kinit admin
Password for admin@IPA.LAB.LAN:

[root@ipa ~]# ipa user-find admin
--------------
1 user matched
--------------
 User login: admin
 Last name: Administrator
 Home directory: /home/admin
 Login shell: /bin/bash
 Principal alias: admin@LAB.LAN
 UID: 1863200000
 GID: 1863200000
 Account disabled: False
----------------------------
Number of entries returned 1
----------------------------
```

### Configure a system as an IPA client

#### Install needed package

```bash
[root@nas01 ~]# yum install freeipa-client
```

#### Verify FQDN configuration from client

```bash
[root@nas01 ~]# hostname -f
nas01.lab.lan
```

#### Join system with ipa-client:

```bash
[root@nas01 ~]# ipa-client-install --mkhomedir
DNS discovery failed to determine your DNS domain
Provide the domain name of your IPA server (ex: example.com): lab.lan
Provide your IPA server name (ex: ipa.example.com): ipa.lab.lan
The failure to use DNS to find your IPA server indicates that your resolv.conf file is not properly configured.
Autodiscovery of servers for failover cannot work with this configuration.
If you proceed with the installation, services will be configured to always access the discovered server for all operations and will not fail over to other servers in case of failure.
Proceed with fixed values and no DNS discovery? [no]: yes
Client hostname: nas01.lab.lan
Realm: LAB.LAN
DNS Domain: lab.lan
IPA Server: ipa.lab.lan
BaseDN: dc=lab,dc=lan

Continue to configure the system with these values? [no]: yes
Synchronizing time with KDC...
Attempting to sync time using ntpd. Will timeout after 15 seconds
Unable to sync time with NTP server, assuming the time is in sync. Please check that 123 UDP port is opened.
User authorized to enroll computers: admin
Password for admin@LAB.LAN:
Successfully retrieved CA cert
 Subject: CN=Certificate Authority,O=LAB.LAN
 Issuer: CN=Certificate Authority,O=LAB.LAN
 Valid From: 2018-02-24 17:02:55
 Valid Until: 2038-02-24 17:02:55

Enrolled in IPA realm LAB.LAN
Created /etc/ipa/default.conf
New SSSD config will be created
Configured sudoers in /etc/nsswitch.conf
Configured /etc/sssd/sssd.conf
Configured /etc/krb5.conf for IPA realm LAB.LAN
trying https://ipa.lab.lan/ipa/json
[try 1]: Forwarding 'schema' to json server 'https://ipa.lab.lan/ipa/json'
trying https://ipa.lab.lan/ipa/session/json
[try 1]: Forwarding 'ping' to json server 'https://ipa.lab.lan/ipa/session/json'
[try 1]: Forwarding 'ca_is_enabled' to json server 'https://ipa.lab.lan/ipa/session/json'
Systemwide CA database updated.
Adding SSH public key from /etc/ssh/ssh_host_rsa_key.pub
Adding SSH public key from /etc/ssh/ssh_host_ecdsa_key.pub
Adding SSH public key from /etc/ssh/ssh_host_ed25519_key.pub
[try 1]: Forwarding 'host_mod' to json server 'https://ipa.lab.lan/ipa/session/json'
Could not update DNS SSHFP records.
SSSD enabled
Configured /etc/openldap/ldap.conf
No SRV records of NTP servers found. IPA server address will be used
NTP enabled
Configured /etc/ssh/ssh_config
Configured /etc/ssh/sshd_config
Configuring lab.lan as NIS domain.
Client configuration complete.
The ipa-client-install command was successful
```

#### Configure ntp to use the ipa ntp server

```bash
[root@nas01 ~]# systemctl stop ntpd
[root@nas01 ~]# ntpdate -s ipa.lab.lan
[root@nas01 ~]# systemctl start ntpd
[root@nas01 ~]# ntpq -p
 remote refid st t when poll reach delay offset jitter
 ==============================================================================
 *ipa.lab.lan 212.45.144.88 3 u 89 128 377 0.154 11.181 1.364
```
