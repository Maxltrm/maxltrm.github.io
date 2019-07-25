---
layout: post
title:  "Perform HP ILO password reset without knowing the old one"
date:   2017-04-20 19:09:54 +0200
categories: linux notes
---

#### Install hponcfg tool on Linux host

#### From the host create a file with the configurations to supply to the ilo

```bash
vi reset_ilo.txt

<ribcl VERSION="2.0">

 <login USER_LOGIN="Administrator" PASSWORD="anypwd">

 <user_INFO MODE="write">

 <mod_USER USER_LOGIN="Administrator">

 <password value="newpwd"/>

 </mod_USER>

 </user_INFO>

 </login>

</ribcl>
```

#### Reset password:

```bash
hponcfg -f reset_ilo.txt -l log.txt

HP Lights-Out Online Configuration utility

Version 4.6.0 Date 09/28/2015 (c) Hewlett-Packard Company, 2015

Firmware Revision = 2.15 Device type = iLO 2 Driver name = hpilo

Script succeeded
```

#### The ILO will be accessible with the new **newpwd**

If you want to configure other parameters:

```bash
<ribcl VERSION="2.0">
 <login USER_LOGIN="Administrator" PASSWORD="boguspassword">
 <user_INFO MODE="write">
 <mod_USER USER_LOGIN="Administrator">
 <password value="verysurepwd"/>
 </mod_USER>
 </user_INFO>
 <RIB_INFO MODE="WRITE" >
 <MOD_NETWORK_SETTINGS>
 <IP_ADDRESS VALUE = "x.x.x.x"/>
 <SUBNET_MASK VALUE = "x.x.x.x"/>
 <GATEWAY_IP_ADDRESS VALUE = "x.x.x.x"/>
 <PRIM_DNS_SERVER value = "x.x.x.x"/>
 <DHCP_ENABLE VALUE = "N"/>
 </MOD_NETWORK_SETTINGS>
 </RIB_INFO>
 </login>
 </ribcl>
```
