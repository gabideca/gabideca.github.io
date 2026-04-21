---
title: "Tutorial 1 - Setting up a test environment with QEMU and libvirt"
date: 2026-02-27 19:37:45 -0300
categories: [Kernel, Tutoriais]
tags: [linux, arm64, qemu, libvirt, kernel]
---

This marks now the first official post in relation to the MAC0470 - Free Software Development course.

## Disclaimers
The more keenly eyed reader will notice that the dates from this post and the previous one are the same.
This is no accident, and is the direct consequence of what was described on the latter.

The process of tinkering, testing and setting up my Linux environment took a considerable time, so long so that I was not able to tackle tutorials 1 and 2 "with my own hands" if one will.

Conveninely enough, however, the subject is setup in groups from the get-go and I was lucky enough to have a very cooperative and communicative group I would like to introduce: André Jun Hirata, who you might remember, and Guilherme Santos Gabriel.

The post you are about to read is a product of mine and Guilherme's hands-on experience with [tutorial 1](https://flusp.ime.usp.br/kernel/qemu-libvirt-setup/) with a few sprinkles of Adnré's takes.
This will be the case for this post and the following one, but from the third tutorial onwards my own experience will be described.

For all the blogposts you are (hopefully) about to read, I will try to point the topics covered in the post's main theme,the biggest obstacles faced and how we managed to solve them.

## What The Tutorial Covers
This tutorial guides the user through the setup of a Linux-QEMU environment capable of compiling and installing custom kernels from source code.
In practical terms: 
* Creating scripts
* Defining environment variables
* Installing Linux images and setting up libvirt VMs
* Setting up SSH
* Host-local file sharing

# Main Issues
## Issue 1 - Virtual Filesystem
This tutorial involved downloading and isntalling a QEMU image. Naturally, the first issue that came up was a version-mismatch related:
At a certain point, the guide asks for the following command to be ran

```bash
virt-filesystems --long --human-readable --all --add "${VM_DIR}/base_arm64_img.qcow2"
```

Which in our case returned the following error

```bash
libguestfs: error: /usr/bin/supermin exited with error status 1.
To see full error messages you may need to enable debugging.
Do:
  export LIBGUESTFS_DEBUG=1 LIBGUESTFS_TRACE=1
and run the command again.
```

We concluded that the issue was related to the `libguestfs` library, specifically the `supermin` component whose main role is to setup a temporary appliance.

## Solution 1 - Guestfish
We applied `guestfish` directly, which circumvented the issue and listed the image partitions

```bash
sudo guestfish --ro -a "${VM_DIR}/base_arm64_img.qcow2" run : list-partitions
```

## Issue 2: VM IP switching between initialisations
By default, libvirt's `default` network sets up IPs thorugh DHCP, dynamically setting IPs on every initialisation, which completely breaks any prior SSH configurations.

## Solution 2 - Associate the VM's MAC address to a static IP.

Firstly, we found the MAC address:

```bash
sudo virsh dumpxml arm64 | grep "mac address"
```

Then, `default` network reconfiguration:

```bash
sudo virsh net-edit default
```

Changing the `<dhcp>` `host` entry:

```xml
<host mac="XX:XX:XX:XX:XX:XX" name="arm64" ip="192.168.122.100"/>
```

And finally, resetting the network:

```bash
sudo virsh net-destroy default
sudo virsh net-start default
```
