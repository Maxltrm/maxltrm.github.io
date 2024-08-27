---
layout: post
title:  "Home NAS downsizing"
date:   2024-08-27 12:50:54 +0100
categories: notes how-to raspberry-pi
---

## **Home NAS downsizing**

### Introduction

Back in 2020, I decided to de-Google my life and I started building a home NAS solution, precisely the ```2020-08-04 17:23``` my home cloud was born:

```bash
# stat / | grep Birth | awk -F'Birth:' '{ print $2 }'
 2020-08-04 17:23:51.000000000 +0100
 ```

### Initial configuration

The hardware was mostly cheap and second-hand:

1. **CPU**: AMD Ryzen 5 1600 Six-Core Processor
1. **Memory**: 2 x 8GB DDR4, Configured Memory Speed: 2400 MT/s
1. **Mobo**: Gigabyte B450M H
1. **NIC**: Realtek Semiconductor Co., Ltd. RTL8111/8168/8211/8411 PCI Express Gigabit Ethernet Controller

The software I used at the time was:

 1. **Openmedivault**: To have a UI for the disks management.
 1. **Nextcloud**: So I could use their app on my devices to sync my data.
 1. **Photoprism**: To have an AI-powered photos search tool since I found almost impossible to view and find my photos using the nexcloud apps.

I have 3 x [WD Red 2TB disks](https://www.westerndigital.com/en-ie/products/internal-drives/wd-red-plus-sata-3-5-hdd?sku=WD20EFPX){:target="_blank"} configured in a soft RAID 5 using mdadm:

```bash
mdadm -v --detail --scan /dev/md/nas 
ARRAY /dev/md/nas level=raid5 num-devices=3 metadata=1.2 name=nas:nas UUID=8fd4c51f:2168ce85:e6a91bfb:f2d97183
 devices=/dev/sda,/dev/sdd,/dev/sde
```

The boot disk is configured in a RAID 1 using 2 x cheap 120GB SSDs:

```bash
mdadm -v --detail --scan /dev/md1
ARRAY /dev/md1 level=raid1 num-devices=2 metadata=1.2 name=nas:0 UUID=8d8b6ab0:e5808d8c:3309ef43:6cddd059
 devices=/dev/sdf1,/dev/sdg1

 df -h /
Filesystem      Size  Used Avail Use% Mounted on
/dev/md1p1       47G   27G   18G  61% /
```
<br>
![](/assets/images/old-nas.jpeg)

### How it's going

After four years the solution has worked quite well, I have managed to keep the entire software stack up-to-date, I started with Openmediavault 5 and I managed to upgrade all the way up to 7 without reinstalling.

I have lost a few disks occasionally (one at a time otherwise I would not have been able to rebuild the raid as it is a RAID 5), and I have always managed to replace faulty disks without losing data.

### Downsides of this configuration

I'm mostly happy with this configuration, the biggest downside right now is the hardware I use.

The system is running on an old Ryzen 5 1600, the CPU is still quite performant, especially seen the workload I use it for, but there are some [well-known issues when using it with Linux](https://www.google.com/search?q=ryzen+5+1600+linux+max_cstate&client=firefox-b-d&sca_esv=20ef6b07cd0fcad8&sca_upv=1&sxsrf=ADLYWIJetnGpqbIRdU6u210LHOlT1NgO5g%3A1724761858354&ei=AsfNZoerFcK1hbIP5Yy9kAE&ved=0ahUKEwiHtI7PlpWIAxXCWkEAHWVGDxIQ4dUDCA8&uact=5&oq=ryzen+5+1600+linux+max_cstate&gs_lp=Egxnd3Mtd2l6LXNlcnAiHXJ5emVuIDUgMTYwMCBsaW51eCBtYXhfY3N0YXRlMgUQIRigAUiiElBWWI0RcAF4AJABAJgBlwGgAfgJqgEDMi45uAEDyAEA-AEBmAIMoAKXC8ICBxAjGLADGCfCAg4QABiABBiwAxiGAxiKBcICCxAAGIAEGLADGKIEwgIEECMYJ8ICCBAAGIAEGKIEwgIFECEYnwXCAgcQIRigARgKmAMAiAYBkAYJkgcEMS4xMaAHyyU&sclient=gws-wiz-serp){:target="_blank"}.

To prevent freezes I had to set the max_cstates to 1, which means that the only [processor states](https://en.wikipedia.org/wiki/ACPI#Processor_states){:target="_blank"} available are:

1. **C0**: Operating state.
1. **C1**: The processor is not executing instructions, but can return to an executing state essentially instantaneously.

On the other hand, setting the max cstate to 1 prevents any power-saving features of the CPU, and this is the biggest problem for me at the moment, this solution is too expensive to run.

The second biggest issue is the physical space the hardware occupies in my house.

Last but not least, the whole thing is too noisy.

### Plans for the future

I want a more energy-efficient solution and can accept some compromises in terms of performance.

I recently put my hands on a raspberry pi 5 and watched this [Jeff Geerling's video](https://www.youtube.com/watch?v=l30sADfDiM8){:target="_blank"}, I'll probably attempt to downsize my current solution and try to migrate it to a raspberry pi 5 powered NAS but first I want to measure the actual operating costs of the current solution, its performance and put them in writing so to be able to compare the two solutions analitically. 

I'll collect a month of power utilization data do some benchmarks and take some notes about it, then I'll start engineering the new solution and do the same analysis to be able to compare the two solutions.


