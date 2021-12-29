---
layout: post
title:  "Configure podman on Gentoo"
date:   2021-11-28 17:43:54 +0200
categories: linux notes
---

Here are some notes on how to install podman on Gentoo, refer to the package [documentation](https://wiki.gentoo.org/wiki/Podman) for further details, for example how to configure your kernel.

#### Set required USE flags

```
echo ">=app-emulation/podman rootless btrfs" > /etc/portage/package.use/podman
```

#### Install podman

```
emerge --ask --oneshot app-emulation/crun app-emulation/podman
```

#### Enable config files

```
cp -p /etc/containers/policy.json.example /etc/containers/policy.json
cp -p /etc/containers/registries.conf.example /etc/containers/registries.conf
```

To date my registries.conf only contains:

```
unqualified-search-registries = ["registry.fedoraproject.org", "registry.access.redhat.com", "docker.io", "quay.io"]
```

## Configure subuid and subgid

Check that subuid and subgid are properly configured, my user was set as ```<username>:0:0``` so it was not allowed to map any user ids from its namespace into child namespaces.

set subuid and subgid as follows:

```
cat /etc/subuid
<username>:100000:65536
cat /etc/subgid
<username>:100000:65536
```

or you would get the following error when trying to run rootless containers:

```
Error: cannot setup namespace using newuidmap: exit status 1
```

#### Set user permissions to rwx for newuidmap and newgidmap

```
chmod 4755 /usr/bin/newuidmap
chmod 4755 /usr/bin/newgidmap
```
