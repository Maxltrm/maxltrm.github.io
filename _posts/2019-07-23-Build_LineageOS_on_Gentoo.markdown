---
layout: post
title:  "Build LineageOS on Gentoo"
date:   2019-07-23 14:09:54 +0200
categories: android how-to
---

I have recentlly received an old [NVIDIA Shield K1 tablet](https://www.gsmarena.com/nvidia_shield_k1-7832.php){:target="_blank"} and, due to few hardware capacity of that device that make it quite useless in 2019, I have decided to use it like an ePub reader and for basic tasks with a light ROM.

After doing a bit of research I have read on [lineage wiki](https://wiki.lineageos.org/devices/shieldtablet/){:target="_blank"} that this device was officially supported but no longer mantained, so I have decided to make my own build of lineageOS, here is a guide on how build a lineageOS 15.1 on Gentoo.

### Official build documentation

[Official build doc](https://wiki.lineageos.org/devices/shieldtablet/build){:target="_blank"}

### Build LineageOS 15.1 for shieldtablet on Gentoo

I'm using as profile tagets:

```
default/linux/amd64/17.1/desktop/plasma/systemd
```

#### Define packages use flags 

```bash
echo "sys-libs/ncurses tinfo abi_x86_32 -gpm" >> /etc/portage/package.use/ncurses
echo "sys-libs/readline abi_x86_32" >> /etc/portage/package.use/readline
echo "dev-java/icedtea:8 headless-awt -alsa -cups -gtk" >> /etc/portage/package.use/icedtea
echo "sys-libs/zlib abi_x86_32" >> /etc/portage/package.use/zlib
echo "app-arch/lz4 abi_x86_32" >> /etc/portage/package.use/lz4
```

use flags meaning:

* **abi_x86_32**: build 32-bit (x86) libraries
* **tinfo**: Link against libtinfo from sys-libs/ncurses for use in cuda and other packages

#### Install needed packages

```bash
emerge -av app-arch/lz4 app-arch/lzop media-gfx/imagemagick media-gfx/pngcrush dev-util/android-tools dev-util/android-sdk-update-manager sys-devel/bc net-misc/curl dev-vcs/git media-gfx/imagemagick sys-libs/ncurses:0 sys-libs/ncurses:5 sys-libs/readline sys-libs/zlib app-arch/lz4 media-libs/libsdl x11-libs/wxGTK:3.0 app-arch/lzop media-gfx/pngcrush sys-process/schedtool sys-fs/squashfs-tools app-arch/zip dev-java/icedtea:8
```

#### Set icedtea-8 and python2.7

```bash
eselect java-vm list
   [1]   icedtea-8

eselect java-vm set system 1
eselect python set python2.7
```

#### Create build environment

```bash
mkdir -p ~/bin
mkdir -p ~/android/lineage
```

#### Install repo command

```bash
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
```

#### Put the ~/bin directory in your path of execution

```bash
vim ~/.bashrc

# set PATH so it includes user's private bin if it exists
if [ -d "$HOME/bin" ] ; then
    PATH="$HOME/bin:$PATH"
fi
```

#### Initialize LineageOS respository

```bash
cd ~/android/lineage
repo init -u https://github.com/LineageOS/android.git -b lineage-15.1
```


#### Sync the repository

```bash
repo sync
```

#### Prepare the device-specific code

```bash
cd ~/android/lineage
source build/envsetup.sh
breakfast shieldtablet
```

#### Extract propietary blobs:

To extract the proprietary blobs you need to have an installed LineageOS on the device:

```bash
~/android/lineage/device/nvidia/shieldtablet 
./extract-files.sh
```

otherwise it's possible to extract the blobs from a LineageOS zip file following this guide:

```
https://wiki.lineageos.org/extracting_blobs_from_zips.html
```

#### Turn on ccache caching

```bash
export USE_CCACHE=1
ccache -M 50G
export CCACHE_COMPRESS=1
```

#### Configure jack

Jack is an Android toolchain that compiles Java source into Android dex bytecode, if not configured properly it can fail and run out of memory.

```bash
export ANDROID_JACK_VM_ARGS="-Dfile.encoding=UTF-8 -XX:+TieredCompilation -Xmx4G"
```

#### Start the build

```bash
croot
brunch shieldtablet
```

#### Install your rom from TWRP or sideload

```bash
adb sideload ~/android/lineage/out/target/product/shieldtablet/lineage-15.1-20190723-UNOFFICIAL-shieldtablet.zip
```

### Useful links
[Unlock bootloader](http://developer.download.nvidia.com/mobile/shield/ROM/ST8K1/0_0_0_Factory/HowTo-Flash-Recovery-Image.txt){:target="_blank"}

[Open Source + Binary driver release](http://nv-tegra.nvidia.com/gitweb/?p=manifest/android/binary.git;a=blob_plain;f=README;hb=rel-24-uda-r1.2-partner){:target="_blank"}

[TWRP img download](https://eu.dl.twrp.me/shieldtablet/){:target="_blank"}

[SuperSU](http://www.supersu.com/download){:target="_blank"}
