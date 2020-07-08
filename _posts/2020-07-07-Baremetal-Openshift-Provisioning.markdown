---
layout: post
title:  "UPI Openshift Provisioning"
date:   2020-07-07
categories: Openshift
---

Since I've started exploring OpenShift I built a home lab, here are some notes about the installation of my OpenShift lab environment.

Even though I'm running the nodes cluster on oVirt, I've followed the instructions for the baremetal provisioning because I wanted to go through all the steps, so this is a baremetal UPI deployment.

The cluster nodes are living on an isolated network, there is a pfsense firewall between my clients network and the cluster private network, on that pfsense I've reserved a static IP for each vm and then I've configured a 1:1 NAT to reach the ocp-utils vm from my clients.

I also use the NAT IP as a nameserver on my clients to resolve the openshift routes associated with the services that I'll run on openshift.

#### HLD

![](/assets/ocp-poc.png)

#### VMs

name | vCPU | Memory | IP | Descritpion
--- | --- | --- | ---
ocp-utils | 1 | 4 | 192.168.100.200 | Runs bind9 and haproxy
ocp-bootstrap | 4 | 16 | 192.168.100.201 | Bootstrap node
ocp-control-plane-1 | 4  | 8 | 192.168.100.202 | Control plane node1
ocp-control-plane-2 | 4  | 8 | 192.168.100.203 | Control plane node2
ocp-control-plane-3 | 4  | 8 | 192.168.100.204 | Control plane node3
ocp-compute-1 | 4  | 8 | 192.168.100.205 | Compute node 1
ocp-compute-2 | 4  | 8 | 192.168.100.206 | Compute node 2

#### DNS Configuration

OpenShift requires a DNS server in the environment to provide name resolution to hosts and containers running on the platform, follow the RedHat doc at [link](https://docs.openshift.com/container-platform/4.2/installing/installing_bare_metal/installing-bare-metal.html#installation-dns-user-infra_installing-bare-metal) to configure the DNS server.

named.conf

```
options {
        listen-on port 53 { 127.0.0.1; 192.168.100.200; };
#       listen-on-v6 port 53 { ::1; };
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        recursing-file  "/var/named/data/named.recursing";
        secroots-file   "/var/named/data/named.secroots";
        allow-query     { localhost; 192.168.2.0/24; 192.168.100.0/24; };

        recursion yes;

        dnssec-enable yes;
        dnssec-validation yes;

        /* Path to ISC DLV key */
        bindkeys-file "/etc/named.root.key";

        managed-keys-directory "/var/named/dynamic";

        pid-file "/run/named/named.pid";
        session-keyfile "/run/named/session.key";
};

logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};

zone "." IN {
        type hint;
        file "named.ca";
};

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";
include "/etc/named/named.conf.local";
```

named.conf.local

```
zone "ocp.local" {
    type master;
    file "/etc/named/zones/db.ocp.local"; # zone file path
};

zone "100.168.192.in-addr.arpa" {
    type master;
    file "/etc/named/zones/db.192.168.100";  # 192.168.100.0/24 subnet
};
```

db.ocp.local

```
$TTL    604800
@       IN      SOA     ocp-utils.ocp.local. admin.ocp.local. (
                  5     ; Serial
             604800     ; Refresh
              86400     ; Retry
            2419200     ; Expire
             604800     ; Negative Cache TTL
)

; name servers - NS records
    IN      NS      ocp-utils

; name servers - A records
ocp-utils.ocp.local.          IN      A       192.168.100.200

; OpenShift Container Platform Cluster - A records
ocp-bootstrap.ocp.local.        IN      A      192.168.100.201
ocp-control-plane-1.ocp.local.        IN      A      192.168.100.202
ocp-control-plane-2.ocp.local.         IN      A      192.168.100.203
ocp-control-plane-3.ocp.local.         IN      A      192.168.100.204
ocp-compute-1.ocp.local.        IN      A      192.168.100.205
ocp-compute-2.ocp.local.        IN      A      192.168.100.206

; OpenShift internal cluster IPs - A records
api.lab.ocp.local.    IN    A    192.168.100.200
api-int.lab.ocp.local.    IN    A    192.168.100.200
*.apps.lab.ocp.local.    IN    A    192.168.100.200
etcd-0.lab.ocp.local.    IN    A     192.168.100.202
etcd-1.lab.ocp.local.    IN    A     192.168.100.203
etcd-2.lab.ocp.local.    IN    A    192.168.100.204
console-openshift-console.apps.lab.ocp.local.     IN     A     192.168.100.200
oauth-openshift.apps.lab.ocp.local.     IN     A     192.168.100.200

; OpenShift internal cluster IPs - SRV records
_etcd-server-ssl._tcp.lab.ocp.local.    86400     IN    SRV     0    10    2380    etcd-0.lab
_etcd-server-ssl._tcp.lab.ocp.local.    86400     IN    SRV     0    10    2380    etcd-1.lab
_etcd-server-ssl._tcp.lab.ocp.local.    86400     IN    SRV     0    10    2380    etcd-2.lab
```

db.192.168.100

```
$TTL    604800
@       IN      SOA     ocp-utils.ocp.local. admin.ocp.local. (
                  8     ; Serial
             604800     ; Refresh
              86400     ; Retry
            2419200     ; Expire
             604800     ; Negative Cache TTL
)

; name servers - NS records
    IN      NS      ocp-utils.ocp.local.

; name servers - PTR records
200    IN    PTR    ocp-utils.ocp.local.

; OpenShift Container Platform Cluster - PTR records
201    IN    PTR    ocp-bootstrap.ocp.local.
202    IN    PTR    ocp-control-plane-1.ocp.local.
203    IN    PTR    ocp-control-plane-2.ocp.local.
204    IN    PTR    ocp-control-plane-3.ocp.local.
205    IN    PTR    ocp-compute-1.ocp.local.
206    IN    PTR    ocp-compute-2.ocp.local.
200    IN    PTR    api.lab.ocp.local.
200    IN    PTR    api-int.lab.ocp.local.
```

#### HAProxy Configuration

OpenShift needs a layer-4 load balancer to access:

* Kubernetes API on port 6443
  - Boostrap node and control plane nodes, the boostrap node will be removed once the bootstrap is completed
* Machine Config Server on port 22623
  - Boostrap node and control plane nodes, the boostrap node will be removed once the bootstrap is completed
* HTTPS traffic on port 443:
  - Machines that run the Ingress router pods, worker nodes in my configuration
* HTTP Traffic on port 80
  - Machines that run the Ingress router pods, worker nodes in my configuration

```
# Global settings
#---------------------------------------------------------------------
global
    maxconn     20000
    log         /dev/log local0 info
    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    user        haproxy
    group       haproxy
    daemon

    # turn on stats unix socket
    stats socket /var/lib/haproxy/stats

#---------------------------------------------------------------------
# common defaults that all the 'listen' and 'backend' sections will
# use if not designated in their block
#---------------------------------------------------------------------
defaults
    mode                    http
    log                     global
    option                  httplog
    option                  dontlognull
    option http-server-close
    option forwardfor       except 127.0.0.0/8
    option                  redispatch
    retries                 3
    timeout http-request    10s
    timeout queue           1m
    timeout connect         10s
    timeout client          300s
    timeout server          300s
    timeout http-keep-alive 10s
    timeout check           10s
    maxconn                 20000

listen stats
    bind :9000
    mode http
    stats enable
    stats uri /

frontend ocp_k8s_api_fe
    bind :6443
    default_backend ocp_k8s_api_be
    mode tcp
    option tcplog

backend ocp_k8s_api_be
    balance source
    mode tcp
    server      ocp-bootstrap 192.168.100.201:6443 check
    server      ocp-control-plane-1 192.168.100.202:6443 check
    server      ocp-control-plane-2 192.168.100.203:6443 check
    server      ocp-control-plane-3 192.168.100.204:6443 check

frontend ocp_machine_config_server_fe
    bind :22623
    default_backend ocp_machine_config_server_be
    mode tcp
    option tcplog

backend ocp_machine_config_server_be
    balance source
    mode tcp
    server      ocp-bootstrap 192.168.100.201:22623 check
    server      ocp-control-plane-1 192.168.100.202:22623 check
    server      ocp-control-plane-2 192.168.100.203:22623 check
    server      ocp-control-plane-3 192.168.100.204:22623 check

frontend ocp_http_ingress_traffic_fe
    bind :80
    default_backend ocp_http_ingress_traffic_be
    mode tcp
    option tcplog

backend ocp_http_ingress_traffic_be
    balance source
    mode tcp
    server      ocp-compute-1 192.168.100.205:80 check
    server      ocp-compute-2 192.168.100.206:80 check

frontend ocp_https_ingress_traffic_fe
    bind *:443
    default_backend ocp_https_ingress_traffic_be
    mode tcp
    option tcplog

backend ocp_https_ingress_traffic_be
    balance source
    mode tcp
    server      ocp-compute-1 192.168.100.205:443 check
    server      ocp-compute-2 192.168.100.206:443 check
```

#### Installer Configuration

Create a file called install-config.yaml and paste the following, you can find further details on the RedHat documentation at [link](https://docs.openshift.com/container-platform/4.3/installing/installing_bare_metal/installing-bare-metal.html#installation-bare-metal-config-yaml_installing-bare-metal)

```
apiVersion: v1
baseDomain: ocp.local
metadata:
  name: lab

compute:
- hyperthreading: Enabled
  name: worker
  replicas: 0

controlPlane:
  hyperthreading: Enabled
  name: master
  replicas: 2

networking:
  clusterNetwork:
  - cidr: 10.128.0.0/14 
    hostPrefix: 23 
  networkType: OpenShiftSDN
  serviceNetwork: 
  - 172.30.0.0/16

platform:
  none: {}

fips: false

pullSecret: your secret
sshKey: your ssh public key
```

#### Create Manifest and Ignition files

I attempted the installation a number of times so I wrote a simple playbook to automate the manifest and ignition creation when I had to start from scratch

{% highlight python %}
{% raw %}
---
- hosts: localhost
  connection: local
  vars:
    install_dir: "/root/install_dir"
    webserver_dir: "/var/www/html/okd4"
    bin_path: "/usr/local/bin"
    version: 4.3.5
  tasks:
    - name: Download openshift installer
      get_url:
        url: "https://mirror.openshift.com/pub/openshift-v4/clients/ocp/{{ version }}/openshift-install-linux-{{ version }}.tar.gz"
        dest: "/tmp/"

    - name: Unarchive openshift installer
      unarchive:
        src: "/tmp/openshift-install-linux-{{ version }}.tar.gz"
        dest: "{{ bin_path }}"
        mode: 0755

    - name: Download openshift client
      get_url:
        url: "https://mirror.openshift.com/pub/openshift-v4/clients/ocp/4.3.5/openshift-client-linux-{{ version }}.tar.gz"
        dest: "/tmp/"

    - name: Unarchive openshift client
      unarchive:
        src: "/tmp/openshift-client-linux-{{ version }}.tar.gz"
        dest: "{{ bin_path }}"
        mode: 0755

    - name: Clear install_dir
      file:
        path: "{{ install_dir }}"
        state: absent

    - name: Create install_dir
      file:
        path: "{{ install_dir }}"
        state: directory

    - name: Clear {{ webserver_dir }}
      file:
        path: "{{ webserver_dir }}"
        state: absent

    - name: Create {{ webserver_dir }}
      file:
        path: "{{ webserver_dir }}"
        state: directory
        mode: 0777

    - name: Copy install-config.yaml into install_dir
      copy:
        src: "/root/install-config.yaml"
        dest: "{{ install_dir }}"

    - name: Create Manifest
      shell:  "openshift-install create manifests --dir={{ install_dir }}"

    - name: Make control plane not schedulable
      replace:
        path: "{{ install_dir }}/manifests/cluster-scheduler-02-config.yml"
        regexp: '^\s+(mastersSchedulable:)(.*)$'
        replace: '  \1 false'

    - name: Create ignition files
      shell:  "openshift-install create ignition-configs --dir={{ install_dir }}"

    - name: Copy files in {{ webserver_dir }}
      shell: "cp -R {{ install_dir }}/* {{ webserver_dir }}"

    - name: Download RHCOS
      get_url:
        url: https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/latest/latest/rhcos-4.4.3-x86_64-metal.x86_64.raw.gz
        dest: "{{ webserver_dir }}/rhcos.raw.gz"

    - name: Set permissions
      shell: "chmod -R 777 {{ webserver_dir }}"
{% endraw %}
{% endhighlight %}

#### Bootstrap box initialization

1. boot the box from the rhcos installation iso
2. Press the tab button to append kernel commands
3. Append:

```
coreos.inst=yes coreos.inst.install_dev=sda coreos.inst.image_url=http://192.168.100.200:8080/ocp/rhcos.raw.gz coreos.inst.ignition_url=http://192.168.100.200:8080/ocp/bootstrap.ign
```

#### Masters boxes initialization

1. boot the box from the rhcos installation iso
2. Press the tab button to append kernel commands
3. Append:

```
coreos.inst=yes coreos.inst.install_dev=sda coreos.inst.image_url=http://192.168.100.200:8080/ocp/rhcos.raw.gz coreos.inst.ignition_url=http://192.168.100.200:8080/ocp/master.ign
```

#### Check if the bootstrap is complete

```
[root@ocp-utils ~]# openshift-install --dir=install_dir wait-for bootstrap-complete --log-level debug
DEBUG OpenShift Installer v4.3.5
DEBUG Built from commit 82f9a63c06956b3700a69475fbd14521e139aa1e
INFO Waiting up to 30m0s for the Kubernetes API at https://api.lab.ocp.local:6443...
INFO API v1.16.2 up
INFO Waiting up to 30m0s for bootstrapping to complete...
DEBUG Bootstrap status: complete
INFO It is now safe to remove the bootstrap resources
```

#### Disabled the bootstrap box on haproxy

1. Comment out the bootstrap node in the haproxy configuration
2. Restart haproxy


#### Verify that the control plane is up

```
[root@ocp-utils ~]# export KUBECONFIG=./install_dir/auth/kubeconfig

[root@ocp-utils ~]# oc get nodes
NAME                             STATUS   ROLES    AGE   VERSION
ocp-control-plane-1.ocp.local   Ready    master   16m   v1.16.2
ocp-control-plane-2.ocp.local   Ready    master   16m   v1.16.2
ocp-control-plane-3.ocp.local   Ready    master   16m   v1.16.2
```

We can go ahead with the initialization of the workers nodes

#### Workers initialization

1. boot the box from the rhcos installation iso
2. Press the tab button to append kernel commands
3. Append:

```
coreos.inst=yes coreos.inst.install_dev=sda coreos.inst.image_url=http://192.168.100.200:8080/ocp/rhcos.raw.gz coreos.inst.ignition_url=http://192.168.100.200:8080/ocp/worker.ign
````

During the workers initialization check if there are CSR to approve

```
[root@ocp-utils ~]# oc get csr 
NAME        AGE   REQUESTOR                                                                   CONDITION
csr-49kwg   21m   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   Approved,Issued
csr-7nls7   13s   system:node:ocp-compute-2.ocp.local                                        Pending
csr-7nn7t   89s   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   Approved,Issued
csr-9n7jq   75s   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   Approved,Issued
csr-gqmsv   21m   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   Approved,Issued
csr-ht2j7   21m   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper   Approved,Issued
csr-ljjpq   21m   system:node:ocp-control-plane-1.ocp.local                                  Approved,Issued
csr-mnl4k   20m   system:node:ocp-control-plane-3.ocp.local                                  Approved,Issued
csr-qmhxx   15s   system:node:ocp-compute-1.ocp.local                                        Pending
csr-txmq2   21m   system:node:ocp-control-plane-2.ocp.local                                  Approved,Issued

[root@ocp-utils ~]# oc adm certificate approve $(oc get csr -o name) 
certificatesigningrequest.certificates.k8s.io/csr-49kwg approved
certificatesigningrequest.certificates.k8s.io/csr-7nls7 approved
certificatesigningrequest.certificates.k8s.io/csr-7nn7t approved
certificatesigningrequest.certificates.k8s.io/csr-9n7jq approved
certificatesigningrequest.certificates.k8s.io/csr-gqmsv approved
certificatesigningrequest.certificates.k8s.io/csr-ht2j7 approved
certificatesigningrequest.certificates.k8s.io/csr-ljjpq approved
certificatesigningrequest.certificates.k8s.io/csr-mnl4k approved
certificatesigningrequest.certificates.k8s.io/csr-qmhxx approved
certificatesigningrequest.certificates.k8s.io/csr-txmq2 approved
```

#### Verify that all the nodes are Ready

usually a node is NotReady if it cannot communicate with the control plane

```
[root@ocp-utils ~]# oc get nodes
NAME                             STATUS   ROLES    AGE    VERSION
ocp-compute-1.ocp.local         Ready    worker   44m    v1.16.2
ocp-compute-2.ocp.local         Ready    worker   44m    v1.16.2
ocp-control-plane-1.ocp.local   Ready    master   101m   v1.16.2
ocp-control-plane-2.ocp.local   Ready    master   101m   v1.16.2
ocp-control-plane-3.ocp.local   Ready    master   100m   v1.16.2
```

#### Verify that all the cluster operators are available

```
[root@ocp-utild ~]# oc get clusterversion
NAME      VERSION   AVAILABLE   PROGRESSING   SINCE   STATUS
version             False       True          43m     Working towards 4.3.5: 100% complete, waiting on authentication, console

[root@ocp-utils ~]# oc get clusteroperators
NAME                                       VERSION   AVAILABLE   PROGRESSING   DEGRADED   SINCE
authentication                             4.3.5     True        False         False       34m
cloud-credential                           4.3.5     True        False         False      101m
cluster-autoscaler                         4.3.5     True        False         False      85m
console                                    4.3.5     True        False         False      34m
dns                                        4.3.5     True        False         False      91m
image-registry                             4.3.5     True        False         False      88m
ingress                                    4.3.5     True        False         False      39m
insights                                   4.3.5     True        False         False      96m
kube-apiserver                             4.3.5     True        False         False      93m
kube-controller-manager                    4.3.5     True        False         False      91m
kube-scheduler                             4.3.5     True        False         False      92m
machine-api                                4.3.5     True        False         False      91m
machine-config                             4.3.5     True        False         False      89m
marketplace                                4.3.5     True        False         False      87m
monitoring                                 4.3.5     True        False         False      35m
network                                    4.3.5     True        False         False      97m
node-tuning                                4.3.5     True        False         False      87m
openshift-apiserver                        4.3.5     True        False         False      89m
openshift-controller-manager               4.3.5     True        False         False      94m
openshift-samples                          4.3.5     True        False         False      85m
operator-lifecycle-manager                 4.3.5     True        False         False      92m
operator-lifecycle-manager-catalog         4.3.5     True        False         False      92m
operator-lifecycle-manager-packageserver   4.3.5     True        False         False      89m
service-ca                                 4.3.5     True        False         False      96m
service-catalog-apiserver                  4.3.5     True        False         False      89m
service-catalog-controller-manager         4.3.5     True        False         False      90m
storage                                    4.3.5     True        False         False      88m

[root@ocp-utils ~]# oc get clusterversion
NAME      VERSION   AVAILABLE   PROGRESSING   SINCE   STATUS
version   4.3.5     True        False         32m     Cluster version is 4.3.5
```

#### Configure the internal registry

I want to configure the image registry with the storage backend **emptyDir** for my poc, this is meant for non-production clusters, all images are lost if the image registry is restarted.

**If during the installation an object storage is not provided by the platform where you are deploying OpenShift, OpenShift will marks the imageregistry as Remove** RedHat doc at [link](https://docs.openshift.com/container-platform/4.2/registry/configuring_registry_storage/configuring-registry-storage-baremetal.html)

Set the image-registry as Managed as described [here](https://docs.openshift.com/container-platform/4.3/registry/configuring_registry_storage/configuring-registry-storage-baremetal.html#registry-removed_configuring-registry-storage-baremetal)

The resource to modify is: configs.imageregistry/cluster

```
$ oc patch configs.imageregistry.operator.openshift.io cluster --type merge --patch '{"spec":{"storage":{"emptyDir":{}}}}'
```

#### Next steps

* Configure an Identity Provider
