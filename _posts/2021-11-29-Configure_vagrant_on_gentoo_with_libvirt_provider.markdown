---
layout: post
title:  "Configure Vagrant on Gentoo with libvirt provider"
date:   2021-11-29 19:00:54 +0200
categories: linux notes
---

As prerequisite for this guide a working [podman]({{ site.baseurl }}{% post_url  2021-11-28-Configure_podman_on_gentoo %}) environment is required.

#### Emerge vagrant

```
emerge --ask vagrant
```

#### Configure .bashrc as follows

```
#https://github.com/vagrant-libvirt/vagrant-libvirt

vagrant(){
  podman run -it --rm \
    -e LIBVIRT_DEFAULT_URI \
    -v /var/run/libvirt/:/var/run/libvirt/ \
    -v ~/.vagrant.d/boxes:/vagrant/boxes \
    -v ~/.vagrant.d/data:/vagrant/data \
    -v ~/.vagrant.d/data:/vagrant/tmp \
    -v $(realpath "${PWD}"):${PWD} \
    -w $(realpath "${PWD}") \
    --network host \
    --entrypoint /bin/bash \
    --security-opt label=disable \
    docker.io/vagrantlibvirt/vagrant-libvirt:latest \
      vagrant $@
}
```

#### Add a libvirt vagrant box

```
$ vagrant box add rockylinux/8 --provider=libvirt
==> box: Loading metadata for box 'rockylinux/8'
    box: URL: https://vagrantcloud.com/rockylinux/8
==> box: Adding box 'rockylinux/8' (v4.0.0) for provider: libvirt
    box: Downloading: https://vagrantcloud.com/rockylinux/boxes/8/versions/4.0.0/providers/libvirt.box
    box: Calculating and comparing box checksum...
==> box: Successfully added box 'rockylinux/8' (v4.0.0) for 'libvirt'!
```

#### Write a Vagrantfile

```
# -*- mode: ruby -*-
# vi: set ft=ruby :

ENV['VAGRANT_DEFAULT_PROVIDER'] = 'libvirt'


Vagrant.configure("2") do |config|

  ##### DEFINE VM #####
  config.vm.define "rocky-01" do |config|
  config.vm.hostname = "rocky-01"
  config.vm.box = "rockylinux/8"
  config.vm.box_check_update = false
  config.vm.network "private_network", ip: "192.168.18.9"
  config.vm.provision :shell, path: "bootstrap.sh"
  config.vm.provider :libvirt do |v|
    v.memory = 2048
    v.cpus = 2
    v.machine_virtual_size = 20
    end
  end
end
```

bootstrap.sh

```
echo ", +" | sudo sfdisk -N 1 /dev/vda --no-reread
partprobe
xfs_growfs -d /dev/vda1
```
