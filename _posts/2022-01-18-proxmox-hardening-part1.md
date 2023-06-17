---
title: proxmox Serverhardening - Part 1 - rpc
author: w3ich3rt
date: 2022-01-20 14:35:00 +0100
categories: [virtualization]
tags: [linux, ssh, config, shell, proxmox]
math: true
mermaid: true
image:
  path: /assets/img/Proxmox.png
  #lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Proxmox Logo
---

In an absolute buying frenzy I rented a new server for my web services, gaming servers and so on. It was immediately clear that I again throw a [proxmox](https://www.proxmox.com/en/ "proxmox") hypervisor on it and since the new server is so potent, I combine all services on it later. :-)
Since the server is hosted by a big german provider this time, the scanner of the BSI of course runs over this IP address range and reminded me directly that I should do some server hardening.

## RPC, RPCBIND (port 111)

`rpc` is (rightly) considered an insecure protocol and of course should not be available to the whole world. But in the default installation of proxmox the service is active and running by default. Now you have a few options 

 - disable the service completely (because it is more or less only necessary for NFS connections)
 - prevent access via proxmox firewall
 - prevent access via the network firewall, behind which the proxmox server hangs.

In my setup the complete disabling is the best choice, so: 

```shell
systemctl disable --now rpcbind.service rpcbind.socket
```
