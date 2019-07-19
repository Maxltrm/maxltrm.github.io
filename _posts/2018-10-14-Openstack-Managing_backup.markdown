---
layout: post
title:  "Openstack - Managing backup"
date:   2018-10-14 16:09:54 +0200
categories: Openstack notes
---

Snpashot and Backup are image of disks created on image store.
Backup and Snapshot are basically same things but they have some differences.
Main difference between backup and snapshot is:

* Snapshot: single point in time
* Backup: many copy
 
### Nova

#### Create a daily instance backup with 3 copy retention

##### with openstack:

{% highlight bash %}
$ openstack server backup create --name <backup_name> --type daily --rotate 3 <server_name>
{% endhighlight %}

##### with nova:

{% highlight bash %}
$ nova backup <server_name> backup daily 3
{% endhighlight %}

##### verify:

{% highlight bash %}
$ openstack image list
$ openstack image show snapshot
$ glance image-list
{% endhighlight %}

#### Create instance snapshot

##### with nova:

{% highlight bash %}
$ nova image-create <server_name> snapshot
{% endhighlight %}

#### Create instance from image:

##### with nova:

{% highlight bash %}
$ nova boot --image backup --flavor <flavor_name> --nic net-name=<net_name> <instance_name>
{% endhighlight %}

### Cinder

#### Create volume snapshot

##### with openstack:

{% highlight bash %}
$ openstack volume list
$ openstack volume snapshot create --volume <volume_name> <snapshot_name>
$ openstack volume snapshot list
{% endhighlight %}

##### with cinder:

{% highlight bash %}
$ cinder list
$ cinder snapshot-createÂ --name-vol-snap <snapshot_name> <volume_name>
{% endhighlight %}

#### Create volume from snapshot

##### with openstack:

{% highlight bash %}
$ openstack volume create --snapshot <snapshot_name> <volume_name>
$ openstack volume list
{% endhighlight %}

##### with cinder:

{% highlight bash %}
$ cinder create --snapshot-id <snapshot_id> --name <volume_name> <size>
$ cinder list
{% endhighlight %}

#### Delete snapshot

##### with openstack:

{% highlight bash %}
$ openstack volume snapshot delete <snapshot_name>
{% endhighlight %}

##### with cinder:

{% highlight bash %}
$ cinder snapshot-delete <snapshot_name>
{% endhighlight %}
