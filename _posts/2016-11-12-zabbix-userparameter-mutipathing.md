---
title: Zabbix-Userparameter - checking the multipathing
author: w3ich3rt
date: 2016-11-12 13:00:00 +0100
categories: [Zabbix]
tags: [linux, monitoring, permissions, shell, zabbix]
math: true
mermaid: true
image:
  path: /assets/img/zabbix.png
  #lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Zabbix Logo  
---

Here are some "quick & dirty" working userparameter for Zabbix :), they check:

- how many paths per device are available, </li>
- if some of that paths are in a faulty state</li>

## Number of paths

```shell
UserParameter=Count.Multipath.Devices, echo $(( $(echo 'show paths' | sudo /sbin/multipathd -k | grep ready | wc -l) / $(echo 'show maps' | sudo /sbin/multipathd -k | grep dm | wc -l) ))
```

this mix of commands uses the `multipathd -k` and gets the paths and maps. Is the amount of `paths / maps = 4` we assume that all 4 paths for each device is ok.

## Checking if some is faulty

```shell
UserParameter=Check.Failed.Path, echo 'show paths' | sudo /sbin/multipathd -k | awk '/failed/||/faulty/||/shaky/||/offline/ {print $2,$5,$6,$7}'|wc -l
```

We pipe the output of `echo 'show path' | multipathd -k` in an `awk` and check if the following failure is in the returnvalue

- <strong>shaky / faulty</strong> If a path connected and ready for I/O is the status "online", otherwise the status is faulty or shaky.
- <strong>failed</strong> is a DM down, its marked as "failed". It means the same like "faulty".
- <strong>offline</strong> Is a SCSI Device deactivated, the Online_Status changed to offline.

The returnvalue of this script is 0, when everything is ok. Is the returnwert something else, you have to check the multipathing.

## Trigger

The trigger in zabbix react to the specific values and because of that, the escalation starts.

## Berechtigungen

To use multipathd in this way, you have to configure some permissions in sudo to get the job done. Take a look to this article [SUDO Settings for Zabbix](https://www.weichert.it/en/sudo-settings-for-zabbix/).
