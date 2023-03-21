---
title: SUDO Settings for Zabbix
author: w3ich3rt
date: 2016-11-13 13:00:00 +0100
categories: [Zabbix]
tags: [linux, monitoring, permissions, shell, sudo, zabbix]
math: true
mermaid: true
image:
  path: /assets/img/zabbix.png
  #lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Zabbix Logo  
---

To use tools like `multipath` in zabbix, you have to check the sudo-settings for the zabbixuser.
Use `visudo` to change the permissions for zabbix.
Zabbix will not use a Terminal `tty`, so you should deactivate that in sudo... You can do this by comment the line `Defaults requiretty` out, or you put that line `Defaults:zabbix !requiretty` into the `visudo`-configfile. If you take method <strong>one</strong>, it is a globalsetting and no connection needs a `tty`. Method <strong>two</strong> will only deactivate that for the `zabbix` user. I recommend method two!
Now you can allow the commands you want to use with zabbix. According to the maxim: "**as little as possible, as much as necessary**" . So you can use a line like that:

```shell
zabbix ALL=NOPASSWD: /sbin/multipathd -k
```

If you want to use more Tools/Commands or even acces to files, you can put them after the command above, comma seperated.
