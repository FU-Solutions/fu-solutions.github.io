---
title: proxmox Serverhardening - Part 2 - rpc
author: w3ich3rt
date: 2022-01-25 14:35:00 +0100
categories: [virtualization]
tags: [linux, ssh, config, shell, proxmox]
math: true
mermaid: true
image:
  path: /assets/img/Proxmox.png
  #lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Proxmox Logo
---

# Part 2 - fail2ban on a proxmox host

Part 1 was about the insecure service (`rpc`) which is rolled out in the default `proxmox` installation. You can find the article [here](https://www.fu-solutions.de/en/2022/01/ulli/administration-en/proxmox-serverhardening-part-1-rpc/ "here"). In this article we want to further secure our proxmox server and slow down bruteforce attacks at least once by temporarily blocking them.

## How fail2ban works

`fail2ban` checks in logs for *failed logins*. These are usually filtered and monitored via regex. If a user tries to log in and uses the wrong password, the *counter* for the corresponding IP goes up. If the counter reaches a configurable threshold, fail2ban will create a temporary (time is also configurable) `iptables` entry and block access to the corresponding service.

## Installing fail2ban

Installation is as simple as configuration.

```shell
apt install fail2ban
```

I'm going to assume that you are using Debian.... of course the package is available in all major distros :-) .

After that, you should briefly check if the service is already in autostart:

```shell
systemctl status fail2ban.service
```

## Configuration of fail2ban
`fail2ban` comes with a set of automatic modules that check for failed logins. Unfortunately, there is currently no module for proxmox in the default. But that doesn't matter, the configuration is explained in the official documentation and is really simple.

In the (to be created) `/etc/fail2ban/jail.d/proxmox.conf` we add the following code snippet.
Here we specify the corresponding services and the logfile. Here `maxretry` defines the number of failed logins and `bantime` the time the IP will be blocked afterwards.

```shell
[proxmox]
enabled = true
port = https,http,8006
filter = proxmox
logpath = /var/log/daemon.log
maxretry = 3
# 1 hour
bantime = 3600
```
Now we have to create a configfile for the filter. For this we create the file `/etc/fail2ban/filter.d/proxmox.conf` and add the following content:

```shell
[definition]
failregex = pvedaemon\[.*authentication failure; rhost=&lt;HOST&gt; user=.* msg=.*
ignoreregex =.
```
Now the service `fail2ban` must be restarted briefly (`systemctl restart fail2ban`).
In the logfile `/var/log/fail2ban.log` now also the *fail* `proxmox` should show up:
```shell
2021-03-03 17:45:32,928 fail2ban.jail [49691]: INFO Jail 'sshd' started
2021-03-03 17:45:32,928 fail2ban.jail [49691]: INFO Jail 'proxmox' started
```

## Test

If you want to test the configuration once, you can do it easily via the following command.

```shell
fail2ban-regex /var/log/daemon.log /etc/fail2ban/filter.d/proxmox.conf

Running tests
=============

Use failregex filter file : proxmox, basedir: /etc/fail2ban
Use log file : /var/log/daemon.log
Use encoding : UTF-8


Results
=======

Failregex: 0 total

Ignoreregex: 0 total

Date template hits:
|- [# of hits] date format
| [370] {^LN-BEG}(?:DAY )?MON Day %k:Minute:Second(?:\.Microseconds)?(?: ExYear)?
`-

Lines: 370 lines, 0 ignored, 0 matched, 370 missed
[processed in 0.01 sec]

Missed line(s): too many to print.  Use --print-all-missed to print all 370 lines
```

## iptables

In the `iptables`, an IP ban looks like this:

```shell
-A f2b-sshd -s IP.AD.DR.ESS/32 -j REJECT --reject-with icmp-port-unreachable
```

Of course, this can be resolved via this as well. Simply delete the `iptables` rule.
