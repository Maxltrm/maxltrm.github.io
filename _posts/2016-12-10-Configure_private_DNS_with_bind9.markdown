---
layout: post
title:  "Configure private DNS with bind9"
date:   2016-12-10 18:09:54 +0200
categories: linux notes
---

BIND9 is the most used Domain Name System in UNIX environments, a DNS server contains a database of IP addresses and the associated hostnames.

A DNS permits to:

* Resolve an hostname in an IP address - lookup
* Resolve an IP address in an hostname – reverse lookup

### Configure a simple private DNS server

#### My Environment:

| Hostname| IP | OS | Domain |
|:--------|:-------:|:-------:|--------:|
| dns01 |  10.10.10.14/28 | Centos 7.4 | lab.lan |
| ipa   |  10.10.10.13/28 | Centos 7.4 | lab.lan |
| nas01 |  10.10.10.12/28 | Centos 7.4 | lab.lan |
| web01 |  10.10.10.11/28 | Centos 7.0 | lab.lan |
| web02 |  10.10.10.10/28 | Centos 7.0 | lab.lan |
|----

#### Install needed packages

```bash
yum install bind bind-utils -y
```

#### Add trusted DNS client to /etc/named.conf

```bash
 vim /etc/named.conf
 acl "trusted" { 10.10.10.0/28;  };
```

#### Add the IP of the DNS server in listen-on port and comment the listen on IPV6

```bash
  vim /etc/named.conf
  listen-on port 53 { 127.0.0.1; 10.10.10.14; }; 
  #listen-on-v6 port 53 { ::1; };
```

#### Allow the queries to the servers present in the acl trusted previously created

```bash
  vim /etc/named.conf
  allow-query { trusted; };
```

#### Include named.conf.local in /etc/named.conf:

```bash
  vim /etc/named.conf
  include "/etc/named/named.conf.local";
```

#### Configure forward zone and reverse zone

```bash
  vim /etc/named/named.conf.local
  zone "lab.lan" {
    type master;
    file "/etc/named/zones/db.lab.lan";
  };
  zone "10.10.10.in-addr.arpa" {
    type master;
    file "/etc/named/zones/db.10.10.10";
  };
```

#### Create the forward zone file

```bash
vim /etc/named/zones/db.lab.lan
$TTL    604800
@       IN      SOA     dns1.lab.lan. admin.lab.lan. (
              3         ; Serial
             604800     ; Refresh
              86400     ; Retry
            2419200     ; Expire
             604800 )   ; Negative Cache TTL

    IN      NS      dns1.lab.lan.

dns1.lab.lan.          IN      A       10.10.10.14
ipa.lab.lan.           IN      A       10.10.10.13
nas01.lab.lan.         IN      A       10.10.10.12
web01.lab.lan.         IN      A       10.10.10.11
web02.lab.lan.         IN      A       10.10.10.10
```

**SOA** = Start Of Authority record, every domain must have a SOA record, whose value represent:

* The primary name server of the domain
* The responsible of the domain admin.dnsimple.com.
* A timestamp that changes every time the domain is updated
* The number of seconds after which the zone updates
* The number of seconds after which the zone updates following an error
* The number of seconds after which the zone is no longer considered authoritative
* The negative cache TTL.

**NS** = Name servers records, an NS record allows you to assign a subdomain to a set of name servers.

**A** = An A record, permit to map an IP address to an FQDN.

#### Create the reverse zone file

```bash
vim /etc/named/zones/db.10.10.10
$TTL    604800
@       IN      SOA     dns1.lab.lan. admin.lab.lan. (
                              3         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
      IN      NS      dns1.lab.lan.

14   IN      PTR     dns1.lab.lan.
13   IN      PTR     ipa.lab.lan.
12   IN      PTR     nas01.lab.lan.
11   IN      PTR     web01.lab.lan.
10   IN      PTR     web02.lab.lan.
```

**PTR** = Pointer record, is a record that allows you to map an FQDN to an IP, necessary for reverse resolution.

#### Configuration Check

```bash
named-checkconf
named-checkzone ipa.lab.lan /etc/named/zones/db.lab.lan
named-checkzone 168.192.1.in-addr.arpa /etc/named/zones/db.10.10.10
```

#### Configure firewalld & Start service:

```bash
firewall-cmd --add-service=dns; firewall-cmd --reload
systemctl start named; systemctl enable named
```

#### Verify DNS resolution on client

```bash
dig +short ipa.lab.lan A
10.10.10.13
dig +short -x 10.10.10.13
ipa.lab.lan.
```
