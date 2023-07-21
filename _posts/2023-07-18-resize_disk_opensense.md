---
title: Howto - Resize Disk on opensense
author: w3ich3rt
date: 2023-07-18 21:00:00 +0100
categories: [linux]
tags: [opensense, linux, howto, firewall, disk, freebsd, gpart, growfs, resize]
math: true
mermaid: true
image:
  path: /assets/img/linux.png
---

## Problem

Yesterday I had to extend a drive on an opensense firewall. This is installed in a Proxmox environment and there I could extend the harddisk via "resize" quite fast. So I attached 25 GB. Now this adjustment must of course also be carried out in the operating system of the opensense.

### Additional Info

The opensense runs on a FreeBSD and therefore many of the known Linux/Unix tools are not available there or work a bit different. To extend the harddisk you have to do the following.

## Solution

We need to get an overview of the partitions on the disk and, if swap is enabled and at the end of the disk layout, we need to disable swap and delete it. Don't worry, we will reconfigure the swap at the end of the disk.

After "moving" the SWAP area, we can now attach the free space to the partition we want to expand. After that, we still need to resize the filesystem.

> To calculate the new startsector for the swap area, you can use this [excelfile](/assets/downloads/freebsdswapcalc.xlsx).

The commands for this procedure can be found here:

```shell
gpart show #get the available partitions
#after executing show command you gonna get this info
#disk = da0
#freebsd-usf = get id (~3)
#freebsd-swap = get id (~4)
gpart recover da0 #recover the new space added
gpart show #check the new free space is added
#let's remove swap from current location
swapoff -a #disable swap on freebsd
gpart delete -i 4 da0 #freebsd-swap id = 4
gpart show #get free space sector start and size in notepad
#lets move swap to the end of disk (download the calculator down to help and use sector start & free size in calculator)
gpart add -t freebsd-swap -b <Swap Start (Sector)> da0 #get the swap start from calculator and replace in the placeholder.
gpart show #confirm the swap place and size is correct (if something wrong, delete it and recalculate and repeat the previous step)
gpart modify -i 4 -l swapfs da0 #assign label for the new partition for opnsense booting
swapon /dev/gpt/swapfs #start the swapping on the system
#we moved now the swap to the end of disk, let's increase root partition size
gpart resize -i 3 da0 #freebsd-usf id = 3
gpart show #check the resize is done
growfs /dev/gpt/rootfs #scanning the new space added to the partition mount.
exit #exit shell
```
