---
layout: post
title:  "Openstack - Create and connect to an instance"
date:   2018-10-14 15:09:54 +0200
categories: Openstack notes
---

#### Source credential

{% highlight bash %}
# . demo-opernc
{% endhighlight %}

#### Create key pair

{% highlight bash %}
# ssh-keygent -q -N ""
{% endhighlight %}

#### Upload key

{% highlight bash %}
$ openstack keypair create --public-key /root/.ssh/id_rsa.pub demo_key
+-------------+-------------------------------------------------+
| Field       | Value                                           |
+-------------+-------------------------------------------------+
| fingerprint | d7:a8:3a:a4:64:3d:32:72:e5:a3:13:8b:b1:1b:39:22 |
| name        | demo_key                                        |
| user_id     | 5d07489c30b948a5a61053a03aaf7374                |
+-------------+-------------------------------------------------+
{% endhighlight %}

#### Create icmp rule in default security group

{% highlight bash %}
# openstack security group rule create --remote-ip 0.0.0.0/0 --protocol icmp default
{% endhighlight %}

### Create an instance

##### Identify flavor to use
{% highlight bash %}
# openstack flavor list
{% endhighlight %}

##### Identify image to use
{% highlight bash %}
# openstack image list
{% endhighlight %}

##### Identify network to use
{% highlight bash %}
# openstack network list
{% endhighlight %}

##### create instance
{% highlight bash %}
# openstack server create --image cirros --flavor m1.tiny --network private --key-name demo_key vm01
{% endhighlight %}

#### Verify instance state

{% highlight bash %}
# openstack server list
{% endhighlight %}

### Create ssh security group

#### create group
{% highlight bash %}
# openstack security group create ssh
{% endhighlight %}

#### create rule
{% highlight bash %}
# openstack security group rule create --remote-ip 0.0.0.0/0 --dst-port 22 --protocol tcp ssh
+-------------------+--------------------------------------+
| Field             | Value                                |
+-------------------+--------------------------------------+
| created_at        | 2018-10-18T19:01:00Z                 |
| description       |                                      |
| direction         | ingress                              |
| ether_type        | IPv4                                 |
| id                | 1e7de7dc-7778-419c-85ef-9b1606c9f85b |
| name              | None                                 |
| port_range_max    | 22                                   |
| port_range_min    | 22                                   |
| project_id        | 6e28a8a1d73548c9aa439a44803326f1     |
| protocol          | tcp                                  |
| remote_group_id   | None                                 |
| remote_ip_prefix  | 0.0.0.0/0                            |
| revision_number   | 0                                    |
| security_group_id | 2fec3c91-a4c8-442f-bd8f-353befc81a25 |
| updated_at        | 2018-10-18T19:01:00Z                 |
+-------------------+--------------------------------------+
{% endhighlight %}

#### Add security group to instance
{% highlight bash %}
# openstack server add security group vm01 ssh
{% endhighlight %}

### Create floating ip
{% highlight bash %}
# openstack floating ip create public
{% endhighlight %}

### Assign ip to vm
{% highlight bash %}
$ openstack server add floating ip vm01 172.24.4.11
{% endhighlight %}

### Test connection from namespace

{% highlight bash %}
show namespace
ip netns
ping from namespace
ip netns exec <namespace_name> pingÂ 172.24.4.11
ssh from namespace
ip netns exec <namespace_name> ssh -i /root/.ssh/id_rsa cirros@172.24.4.11
{% endhighlight %}
