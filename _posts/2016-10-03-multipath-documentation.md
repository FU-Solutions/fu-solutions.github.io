---
title: Documentation for multipathd
author: w3ich3rt
date: 2016-10-3 14:35:00 +0100
categories: [Linux]
tags: [linux, lun, multipath, multipathd, scsi, shell]
math: true
mermaid: true
image:
  path: /assets/img/linux.png
  #lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Just a logo from fu-solutions
---

# Documentation for MultipathD

The multipathd supports the devicemapper with the configuration of the harddrives which are mounted with more than one path.

## Important Files

The configuration file for the multipathd is the following: - `/etc/multipath.conf` This file holds the configuratipn... -- Blacklisting -- Exceptions -- Assignment -- and so on...
This files would be generated by the multipathd:

- `/etc/multipath/bindings`
> Controls the assignment
- `/etc/multipath/wwids`
> Includes the wwids

## Tools

The tool to build and change the configurationsfiles is `mpathconf`. With it you can modify the `/etc/multipath.conf`. You also could modify the `/etc/multipath.conf` manually. Example:

```shell
defaults {
         udev_dir               /dev
         no_path_retry          1
         polling_interval       10
         path_checker           tur
         path_grouping_policy   group_by_prio
         user_friendly_names    yes
         selector               "round-robin 0"
         max_fds                "4128"
}
blacklist {
        devnode "^(ram|raw|loop|fd|md|dm-|sr|scd|st)[0-9]*"
        devnode "^hd[a-z]"
        devnode "^dcssblk[0-9]*"
        wwid "*"
}
blacklist_exceptions {
        wwid "3600d0230000000000e13955cc3757802"
        wwid "3600d0230000000000e13955cc3757801"
        wwid "3600d0230000000000e13955cc3757800"
        wwid "3600d0230000000000e13955cc3757803"
        wwid "3600d0230000000000e13955cc3757804"
}
multipaths {
        multipath {
                uid 54321
                gid 0
                wwid "3600d0230000000000e13955cc3757800"
                mode 0660
                alias AliasA
        }
        multipath {
                uid 54321
                gid 0
                wwid "3600d0230000000000e13955cc3757801"
                mode 0660
                alias AliasB
        }
        multipath {
                uid 54321
                gid 0
                wwid "3600d0230000000000e13955cc3757802"
                mode 0660
                alias AliasC
        }
        multipath {
                uid 54321
                gid 0
                wwid "3600d0230000000000e13955cc3757803"
                mode 0660
                alias AliasD
        }
        multipath {
                uid 54321
                gid 0
                wwid "3600d0230000000000e13955cc3757804"
                mode 0660
                alias AliasE
        }
}
```

I think this file is selfexplained :)

## Determine WWIDs

To get the WWIDs you can use the command `scsi_id -g -u /dev/sdb`. You will get the WWID for the device sdb. With this little for-loop `for device in /dev/sd*; do echo "Device: ${device} $(scsi_id -g -u ${device})"; done` you will get all WWIDs for the devices. The output should look like that:

```shell
> Device: /dev/sdb 3600d0230000000000e13955cc3757800
> Device: /dev/sdc 3600d0230000000000e13955cc3757801
> Device: /dev/sdd 3600d0230000000000e13955cc3757802
> Device: /dev/sde 3600d0230000000000e13955cc3757803
> Device: /dev/sdf 3600d0230000000000e13955cc3757804
> Device: /dev/sdg 3600d0230000000000e13955cc3757800
> Device: /dev/sdh 3600d0230000000000e13955cc3757801
> Device: /dev/sdi 3600d0230000000000e13955cc3757802
> Device: /dev/sdj 3600d0230000000000e13955cc3757803
> Device: /dev/sdk 3600d0230000000000e13955cc3757804
> Device: /dev/sdl 3600d0230000000000e13955cc3757800
> Device: /dev/sdm 3600d0230000000000e13955cc3757801
> Device: /dev/sdn 3600d0230000000000e13955cc3757802
> Device: /dev/sdo 3600d0230000000000e13955cc3757803
> Device: /dev/sdp 3600d0230000000000e13955cc3757804
> Device: /dev/sdq 3600d0230000000000e13955cc3757800
> Device: /dev/sdr 3600d0230000000000e13955cc3757801
> Device: /dev/sds 3600d0230000000000e13955cc3757802
> Device: /dev/sdt 3600d0230000000000e13955cc3757803
> Device: /dev/sdu 3600d0230000000000e13955cc3757804
```
You have to put this WWIDs into the multipath.conf. You also can use other criterias to set the exceptions, the vendor for example.

## Autoconfig with multipathd

When everything in the configurationsfile is done correctly , the `multipathd` needs to be restartet `service multipathd restart` and the devicemapper should do his magic.
You should check if the `multipathd` is in the autostart. `chkconfig multipathd --list` when not `chkconfig multipathd on`.
