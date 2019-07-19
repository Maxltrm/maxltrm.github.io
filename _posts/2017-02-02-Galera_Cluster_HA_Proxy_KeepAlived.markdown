---
layout: post
title:  "Galera Cluster HA Proxy KeepAlived"
date:   2017-02-02 13:09:54 +0200
categories: database how-to
---

Galera cluster is a synchronous multi-master cluster for MySql.

Here is an example of implementation of a MySql Galera Cluster, balanced by HAProxy reachable from a VIP managed by KeepAlived.

My Environment:

| # | Name | OS | CPU (Mhz) | Memory (GB) |
|:--------|:-------:|:-------:|:-------:|--------:|
| 3 | Galera | Centos 7.3 | 8 (2299.373) | 20 |
| 2 | Haproxy | Centos 7.3 | 8 (2299.373) | 20 |
|----

| Hostname | IP |
|:--------|-------:|
| galera1 | 172.16.100.121 |
| galera2 | 172.16.100.122 |
| galera3 | 172.16.100.123 |
| haproxy | 172.16.100.124 |
| haproxy2 | 172.16.100.125 |
| MYSQL_VIP | 172.16.100.130 |
|----

Before starting, properly configure the hosts files or configure a DNS

    172.16.100.121 galera1
    172.16.100.122 galera2
    172.16.100.123 galera3 

#### [ALL] Add Mariadb repository

    vim /etc/yum.repos.d/mariadb.repo
    
    [mariadb]
    
    name = MariaDB
    baseurl = http://yum.mariadb.org/10.0/centos7-amd64
    gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
    gpgcheck=1

#### [ALL] Install MariaDB Galera Cluster and all related packages

    # yum install MariaDB-Galera-server MariaDB-client rsync galera socat -y 

#### [ALL] Secure MariaDB and set a secure password

    # mysql_secure_installation
    impostare una password per root: Es. verysurepwd

#### [ALL] Create a user dedicated to the SST (State Transfer Snapshot) phase

    # mysql -u root -p
    mysql> DELETE FROM mysql.user WHERE user='';
    mysql> GRANT ALL ON *.* TO 'root'@'%' IDENTIFIED BY ' verysurepwd ';
    mysql> GRANT USAGE ON *.* to sst_user@'%' IDENTIFIED BY ' verysurepwd ';
    mysql> GRANT ALL PRIVILEGES on *.* to sst_user@'%';
    mysql> FLUSH PRIVILEGES;
    mysql> quit

#### [ALL] Create Galera configuration

    # systemctl stop mysql
    # systemctl disable mysql 

Galera1

    cat >> /etc/my.cnf.d/server.cnf << EOF
    binlog_format=ROW
    default-storage-engine=innodb
    innodb_autoinc_lock_mode=2
    innodb_locks_unsafe_for_binlog=1
    query_cache_size=0
    query_cache_type=0
    bind-address=0.0.0.0
    datadir=/var/lib/mysql
    innodb_log_file_size=100M
    innodb_file_per_table
    innodb_flush_log_at_trx_commit=2
    wsrep_provider=/usr/lib64/galera/libgalera_smm.so
    wsrep_cluster_address="gcomm://172.16.100.121, 172.16.100.122, 172.16.100.123"
    wsrep_cluster_name='galera_cluster'
    wsrep_node_address='172.16.100.121'
    wsrep_node_name=' galera1 '
    wsrep_sst_method=rsync
    wsrep_sst_auth=sst_user:verysurepwd
    EOF

Galera2

    cat >> /etc/my.cnf.d/server.cnf << EOF
    binlog_format=ROW
    default-storage-engine=innodb
    innodb_autoinc_lock_mode=2
    innodb_locks_unsafe_for_binlog=1
    query_cache_size=0
    query_cache_type=0
    bind-address=0.0.0.0
    datadir=/var/lib/mysql
    innodb_log_file_size=100M
    innodb_file_per_table
    innodb_flush_log_at_trx_commit=2
    wsrep_provider=/usr/lib64/galera/libgalera_smm.so
    wsrep_cluster_address="gcomm://172.16.100.121, 172.16.100.122, 172.16.100.123"
    wsrep_cluster_name='galera_cluster'
    wsrep_node_address='172.16.100.122'
    wsrep_node_name=' galera2 '
    wsrep_sst_method=rsync
    wsrep_sst_auth=sst_user:verysurepwd
    EOF

Galera3

    cat >> /etc/my.cnf.d/server.cnf << EOF
    binlog_format=ROW
    default-storage-engine=innodb
    innodb_autoinc_lock_mode=2
    innodb_locks_unsafe_for_binlog=1
    query_cache_size=0
    query_cache_type=0
    bind-address=0.0.0.0
    datadir=/var/lib/mysql
    innodb_log_file_size=100M
    innodb_file_per_table
    innodb_flush_log_at_trx_commit=2
    wsrep_provider=/usr/lib64/galera/libgalera_smm.so
    wsrep_cluster_address="gcomm://172.16.100.121, 172.16.100.122, 172.16.100.123"
    wsrep_cluster_name='galera_cluster'
    wsrep_node_address='172.16.100.123'
    wsrep_node_name=' galera3 '
    wsrep_sst_method=rsync
    wsrep_sst_auth=sst_user:verysurepwd
    EOF

#### [GALERA1] Initialize first cluster node

    # /etc/init.d/mysql start --wsrep-new-cluster
    # mysql -uroot -pverysurepwd -e"show status like 'wsrep%'" | egrep "wsrep_local_state_comment|wsrep_incoming_addresses|wsrep_cluster_size|wsrep_ready"
    wsrep_local_state_comment       Synced
    wsrep_incoming_addresses        172.16.100.121:3306
    wsrep_cluster_size      1
    wsrep_ready     ON

#### [GALERA2-3] Start msyql

    # /etc/init.d/mysql start
    # mysql -uroot -pverysurepwd -e"show status like 'wsrep%'" | egrep "wsrep_local_state_comment|wsrep_incoming_addresses|wsrep_cluster_size|wsrep_ready"
    wsrep_local_state_comment       Synced
    wsrep_incoming_addresses        172.16.100.121:3306,172.16.100.122:3306,172.16.100.123:3306
    wsrep_cluster_size      3
    wsrep_ready     ON

The database is now replicated on all the cluster nodes, so it is possible to perform the next step only on a single MariaDB node.

#### Create a user for haproxy stat

    # CREATE USER 'haproxy'@'172.16.100.130'; 

### Configure Haproxy + Keepalived

#### [HAPROXY1 - HAPROXY2] install keepalived and haproxy

    # yum install haproxy keepalived -y 

#### [HAPROXY1 - HAPROXY2] Configure haproxy

    # vim /etc/haproxy/haproxy.cfg
        global
        log         127.0.0.1 local2
        chroot      /var/lib/haproxy
        pidfile     /var/run/haproxy.pid
        maxconn     4000
        user        haproxy
        group       haproxy
        daemon
    
        stats socket /var/lib/haproxy/stats
    
    defaults
        mode                    http
        log                     global
        #option                  httplog
        option                  dontlognull
        option http-server-close
        #option forwardfor       except 127.0.0.0/8
        option                  redispatch
        retries                 3
        timeout http-request    10s
        timeout queue           1m
        timeout connect         10s
        timeout client          1m
        timeout server          1m
        timeout http-keep-alive 10s
        timeout check           10s
        maxconn                 3000
    
    frontend mariadb
            bind 172.16.100.130:3306
            mode tcp
            default_backend galera
    
    backend galera
            balance source
            mode tcp
            option tcpka
            option mysql-check user haproxy
            server node1 172.16.100.121:3306 check weight 1
            server node2 172.16.100.122:3306 check weight 1
            server node3 172.16.100.123:3306 check weight 1

#### Configure keepalived

Haproxy1

    # vim /etc/keepalived/keepalived.conf
    global_defs {
      lvs_id haproxy_DH
    }
    
    vrrp_script chk_haproxy {
      script "killall -0 haproxy"
      interval 2
      weight 2
    }
    
    vrrp_instance VI_1 {
      interface ens192
      state MASTER
      virtual_router_id 51
      priority 101
      unicast_peer {
      haproxy2
      }
      virtual_ipaddress {
        172.16.100.130
      }
      track_script {
        chk_haproxy
      }
    }

Haproxy2

    global_defs {
     lvs_id haproxy_DH_passive
    }
    
    vrrp_script chk_haproxy {
     script "killall -0 haproxy"
     interval 2
     weight 2
    }
    
    vrrp_instance VI_1 {
     interface ens192
     state BACKUP
     virtual_router_id 51
     priority 100
     unicast_peer {
     haproxy1
     }
     virtual_ipaddress {
     172.16.100.130
     }
     track_script {
     chk_haproxy
     }
    }

#### [HAPROXY1 - HAPROXY2]  Configure kernel parameters

The following parameter allows haproxy to bind the port on an IP not yet present on the VM.

    # vim /etc/sysctl.conf
    net.ipv4.ip_nonlocal_bind=1 

#### [HAPROXY1 - HAPROXY2] Start & Enable

    # systemctl start keepalived; systemctl enable keepalived
    # systemctl start haproxy; systemctl enable haproxy 

### Sysbench Benchmark

#### Prepare:

    sysbench --db-driver=mysql --mysql-host=172.16.100.130 --mysql-user=admin --mysql-password=verysurepwd --mysql-db=foo --range_size=100Â  Â --table_size=10000 --tables=2 --threads=1 --events=0 --time=60 --rand-type=uniform /usr/share/sysbench/oltp_read_only.lua prepare 

#### Start benchmark:

* Read_only Benchmark

    sysbench --db-driver=mysql --mysql-host=172.16.100.130 --mysql-user=admin --mysql-password=verysurepwd --mysql-db=foo --range_size=100 --table_size=10000 --tables=2 --threads=1 --events=0 --time=60 --rand-type=uniform /usr/share/sysbench/oltp_read_only.lua run
    
    SQL statistics:
     queries performed:
     read: 297710
     write: 0
     other: 42530
     total: 340240
     transactions: 21265 (354.40 per sec.)
     queries: 340240 (5670.34 per sec.)
     ignored errors: 0 (0.00 per sec.)
     reconnects: 0 (0.00 per sec.)
    
    General statistics:
     total time: 60.0015s
     total number of events: 21265
    
    Latency (ms):
     min: 2.09
     avg: 2.82
     max: 10.09
     95th percentile: 3.43
     sum: 59964.21
    
    Threads fairness:
     events (avg/stddev): 21265.0000/0.00
     execution time (avg/stddev): 59.9642/0.00

* Write_only Benchmark

    sysbench --db-driver=mysql --mysql-host=172.16.100.130 --mysql-user=admin --mysql-password=verysurepwd --mysql-db=foo --range_size=100 --table_size=10000 --tables=2 --threads=1 --events=0 --time=60 --rand-type=uniform /usr/share/sysbench/oltp_write_only.lua run
    sysbench 1.0.9 (using system LuaJIT 2.0.4)
    
    SQL statistics:
     queries performed:
     read: 0
     write: 145749
     other: 126999
     total: 272748
     transactions: 45458 (757.59 per sec.)
     queries: 272748 (4545.56 per sec.)
     ignored errors: 0 (0.00 per sec.)
     reconnects: 0 (0.00 per sec.)
    
    General statistics:
     total time: 60.0012s
     total number of events: 45458
    
    Latency (ms):
     min: 0.93
     avg: 1.32
     max: 11.38
     95th percentile: 1.58
     sum: 59927.30
    
    Threads fairness:
     events (avg/stddev): 45458.0000/0.00
     execution time (avg/stddev): 59.9273/0.00
