---
layout: post
title:  "Openstack - Managing cinder volumes"
date:   2018-10-17 13:09:54 +0200
categories: Openstack notes
---

#### Create volume in a project

##### with openstack:

{% highlight bash %}
$ openstack volume create --os-project-name <project_name> --size <size_in_gb> <volume_name>
$ openstack volume list --os-project-name <project_name>
{% endhighlight %}

##### with nova:

{% highlight bash %}
$ cinder --os-project-name demo create --name <volume_name> <size_in_gb>
{% endhighlight %}

#### Attach and detach volume from server

##### attach with openstack:

{% highlight bash %}
$ openstack --os-project-name <project_name> server list
$ openstack --os-project-name <project_name> server add volume --device /dev/sdx <server_name> <volume_name>
$ openstack --os-project-name <project_name> server show <server_name> | grep vol
{% endhighlight %}

##### detach with openstack:

{% highlight bash %}
$ openstack --os-project-name <project_name> server remove volume <server_name> <volume_name>
{% endhighlight %}

#####  attach with cinder:

{% highlight bash %}
$ cinder --os-project-name <project_name> volume-attach <server_name> <volume_ID> /dev/sdx
{% endhighlight %}

#####  detach with cinder:

{% highlight bash %}
$ nova --os-project-name <project_name> volume-detach <server_name> <volume_ID>
{% endhighlight %}

#### Transfer volume

“Transfer volume” permit to transfer a volume from one owner to another, to do this it’s necessary to create a transfer request, the user recipient will accept the transfer using the transfer request ID and authorization key.

##### with openstack:

Request creator

{% highlight bash %}
$ openstack --os-project-name <project_name> volume transfer request create --name <transfer_name> <volume_name>
{% endhighlight %}

Recipient

{% highlight bash %}
$ openstack volume transfer list
$ openstack volume transfer show <transfer_id>
$ openstack volume transfer request accept <transfer_id> --auth-key <auth_key>
$ openstack volume list
{% endhighlight %}

##### with cinder:

{% highlight bash %}
$ cinder --os-project-name <project_name> transfer-create --name <transfer_name> <volume_name>Â 
$ cinder --os-project-name <project_name> transfer-list
$ cinder transfer show <transfer_id>
$ cinder transfer-accept <transfer_id> <auth_id>
$ cinder list
{% endhighlight %}

#### Resize a volume

##### with openstack:

{% highlight bash %}
$ openstack volume set --size <new_size> <volume_name>
{% endhighlight %}

##### with cinder:

{% highlight bash %}
$ cinder extend <volume_name> <new_size>
{% endhighlight %}

#### Backup & restore

##### with openstack:

{% highlight bash %}
$ openstack volume backup create --name <backup_name> <volume_name>
$ openstack volume backup restore <backup_name> <volume_name>
{% endhighlight %}

##### with cinder:

{% highlight bash %}
$ cinder backup-create --name <backup_name> <volume_name>
$ cinder backup-restore --volume <volume_name> <backup_name>
{% endhighlight %}

#### remove backup

##### with openstack:

{% highlight bash %}
$ openstack volume backup list
$ openstack volume backup delete <backup_name>
{% endhighlight %}

##### with cinder:

{% highlight bash %}
$ cinder backup-delete <backup_name>
{% endhighlight %}
