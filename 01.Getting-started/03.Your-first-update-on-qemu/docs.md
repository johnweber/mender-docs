---
title: Your first update on QEMU
taxonomy:
    category: docs
---

This tutorial will show you how to deploy a rootfs image onto a QEMU machine and verify that the update was successful after reboot. We will use prebuilt images, so you don't have to compile or build Mender.


## Prerequisites

The workstation needs [QEMU](http://wiki.qemu.org/?target=_blank) with ARM processor support installed and minimum 1 GB free memory. QEMU runs on various plattforms and it can easily be installed using package managers.

Debian and Ubuntu:

```
apt-get install qemu-system-arm
```

Red Hat, CentOS and Fedora:

```
yum install qemu-system-arm
```

To verify that QEMU is correctly installed, check its version with:

```
qemu-system-arm -version
```

## Download and unpack prebuilt images 
If you have already [built a Yocto image with Mender](../../Artifacts/Building-Mender-Yocto-image), please move on to the [next section](#run-the-image-in-qemu). If you don't have any images to test, you can download our latest builds which contains the neccessary images for testing. It will also contain images for BeagleBone.

```
mkdir mender
cd mender
wget https://goo.gl/mmJoxs
```

Unpack the tarball:

```
tar -zxvf mmJoxs

  vexpress-qemu/u-boot.elf
  vexpress-qemu/core-image-full-cmdline-vexpress-qemu.sdimg
  vexpress-qemu/core-image-full-cmdline-vexpress-qemu.ext3
  vexpress-qemu/mender-qemu.sh
  beaglebone/core-image-base-beaglebone.ext3
  beaglebone/core-image-base-beaglebone.sdimg
  BUILD
  README
```


## Run the image in QEMU
Run the image in QEMU by running the following commands:

```
cd vexpress-qemu
```
```
/bin/sh mender-qemu.sh
```

This will take you to the login prompt. Above the prompt you should see a welcome message like:

> "Poky (Yocto Project Reference Distro) 2.0.2 vexpress..."

You can login with user *root*. No password is required. 

## Serve a rootfs image for the QEMU machine

To deploy a new rootfs to the QEMU machine, we first start a http server on our workstation to serve the image.

While you are in a directiory with a rootfs image (e.g. ```core-image-full-cmdline-vexpress-qemu.ext3```) on your workstation, start a simple Python webserver that will serve your new image to the QEMU machine.

```
python -m SimpleHTTPServer
```

!!! By default the QEMU machine can reach your workstation on IP address 10.0.2.2 and SimpleHTTPServer starts on port 8000, so your QEMU machine should now be able to access your workstation's directory at ```http://10.0.2.2:8000/```.

## Deploy the new rootfs to the QEMU machine with Mender
To deploy the new image to your QEMU machine, go to its terminal and run the following command:

```
mender -rootfs http://10.0.2.2:8000/core-image-full-cmdline-vexpress-qemu.ext3
```

Mender will download the new image, write it to the inactive rootfs partition and configure the bootloader to boot into it on the next reboot. This should take about 2 minutes to complete.

To run the updated rootfs image, simply reboot your QEMU machine:

```
reboot
```

QEMU should boot into the updated rootfs, and a welcome message like this should greet you:

> "This system has been updated by Mender build 376 compiled on..."

**Congratulations!** You have just deployed your first rootfs image with Mender! If you are happy with the update, you can make it permanent by logging in to the QEMU machine as *root* and running:


```
mender -commit
```

By running this command, Mender will configure the bootloader to persistently boot from this updated rootfs partition.

!!! If we reboot the machine again *without* running ```mender -commit```, it will boot into the previous rootfs partition that is known to be working (where we deployed the update from). This ensures strong reliability in cases where the newly deployed rootfs does not boot or otherwise has issues that we want to roll back from.