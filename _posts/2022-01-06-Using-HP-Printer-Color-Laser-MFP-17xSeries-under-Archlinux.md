---
title: Using HP Printer Color Laser MFP 17x Series under Archlinux
author: w3ich3rt
date: 2022-01-06 14:35:00 +0100
categories: [Linux]
tags: [linux, ssh, config, shell]
math: true
mermaid: true
image:
  path: /assets/img/linux.png
  #lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Linux Logo
---

I bought a HP Printer Color Laser MFP 178nwg for my homeoffice in my new home. Using it under windows is not a problem. You just need to install the HP-Tools and then it should work. But on (arch/manjaro) linux it is quite another story!

## Issues

First of all I tried to install the printer via the Settings-Printer app, but the assistant was not able to get the job done. After some quick research I could gather some  informations, that we should use hplip for this job, but this didn't work either. `hp-setup` (a programm that comes with the hplip package) could not find the printer. But it points me into the direction of this page: [HP Developers Portal](https://developers.hp.com/hp-linux-imaging-and-printing/KnowledgeBase/Troubleshooting/TroubleshootNetwork). I tried the mentioned steps with `hp-makeuri` but it tells me, that there is no device.
After some more research (some googleling) I found out, that there is no modeldefinition in the hplip for my printer - nice! Fortunately there is this [bug-report](https://bugs.launchpad.net/hplip/+bug/1932030) and some cool guy provided an working modeldefinition.

```shell
[hp_color_laser_mfp_178_179]
align-type=0
clean-type=0
color-cal-type=0
copy-type=0
embedded-server-type=1
fax-type=7
fw-download=False
icon=hp_color_laserjet_cp2025.png
io-mfp-mode=1
io-mode=1
io-support=14
job-storage=0
linefeed-cal-type=0
model1=Terrible HP Color Laser MFP 178nwg
monitor-type=0
panel-check-type=0
pcard-type=0
plugin=0
plugin-reason=65
power-settings=0
pq-diag-type=0
r-type=0
r0-agent1-kind=4
r0-agent1-sku=CE311A
r0-agent1-type=4
r0-agent2-kind=4
r0-agent2-sku=CE310A
r0-agent2-type=1
r0-agent3-kind=4
r0-agent3-sku=CE313A
r0-agent3-type=5
r0-agent4-kind=4
r0-agent4-sku=CE312A
r0-agent4-type=6
scan-src=3
scan-type=5
status-battery-check=0
status-dynamic-counters=0
status-type=10
support-released=True
support-subtype=219b2b
support-type=2
support-ver=3.13.11
tech-class=Hbpl1
family-class=PCLM-COLOR
tech-subclass=Color
tech-type=4
usb-pid=332a
usb-vid=3f0
wifi-config=3
```

Just add this definition into the file `/usr/share/hplip/data/models/models.dat`

```shell
sudo vi /usr/share/hplip/data/models/models.dat
```

Now you're able to use `hp-makeuri` to find the properly URI for the Printer to use in CUPS.

```shell
$> hp-makeuri -g 127.0.0.1

HP Linux Imaging and Printing System (ver. 3.21.12)
Device URI Creation Utility ver. 5.0

Copyright (c) 2001-18 HP Development Company, LP
This software comes with ABSOLUTELY NO WARRANTY.
This is free software, and you are welcome to distribute it
under certain conditions. See COPYING file for more details.

hp-makeuri[35008]: debug: Cache miss: hp_color_laser_mfp_178_179
hp-makeuri[35008]: debug: Reading file: /usr/share/hplip/data/models/models.dat
hp-makeuri[35008]: debug: Searching for section [hp_color_laser_mfp_178_179] in file /usr/share/hplip/data/models/models.dat
hp-makeuri[35008]: debug: Found section [hp_color_laser_mfp_178_179] in file /usr/share/hplip/data/models/models.dat
CUPS URI: hp:/net/HP_Color_Laser_MFP_178_179?ip=127.0.0.1
SANE URI: hpaio:/net/HP_Color_Laser_MFP_178_179?ip=127.0.0.1
HP Fax URI: hpfax:/net/HP_Color_Laser_MFP_178_179?ip=127.0.0.1

Done.
```

I changed the IP and you don't need the parameter `-g` it's just for debugoutput.

After that I installed the printer via CUPS Webfrontent (http://localhost:631). But I wasn't able to print something... This is because the Printer does not work with the driver from hplip package. On Archlinux I found the [Arch AUR Package - hpuld](https://aur.archlinux.org/packages/hpuld/). This will provide the needed PPD file. 
Just install the package like `yay -S hpuld`. Then the PPD files will stored in `/opt/hp/printer/share/ppd/`. 

Now we can reconfigure the Printer a last time! Go to the settings from Manjaro and select Printer.
Choose the printer and select `Printer Details` and in the popup menu, select `Install PPD File...`. Load the correct file for your printer. I needed to choose `/opt/hp/printer/share/ppd/HP_Color_Laser_MFP_17x_Series.ppd`. 

Now I was able to print! I Hope this article helps you, if you are in some trouble with your printer as well.

### Links

 - [HP Developers Portal - hp-makeuri](https://developers.hp.com/hp-linux-imaging-and-printing/KnowledgeBase/Troubleshooting/TroubleshootNetwork)
 - [Arch AUR Package - hpuld](https://aur.archlinux.org/packages/hpuld/)
 - [HPLIP - Bugreport](https://bugs.launchpad.net/hplip/+bug/1932030)
