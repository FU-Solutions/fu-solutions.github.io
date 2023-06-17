---
title: CentOS / RHEL - Changing the PHP version
author: w3ich3rt
date: 2021-12-06 14:35:00 +0100
categories: [Linux]
tags: [linux, ssh, config, shell]
math: true
mermaid: true
image:
  path: /assets/img/linux.png
  #lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Linux Logo
---

I was just allowed to update the PHP version on a web server and would like to share the procedure with you here.
With this procedure, I was able to carry out an update within the 7.x releases without any problems. I have also carried out a test update from version 7.4 to PHP8.1. The test page still works without any problems.

The following commands have been executed on a `CentOS` 8 stream installation and will work on all RHEL-based systems or with the package manager `dnf`.

## Installing the necessary repos

First we need to install the necessary repos for the update. To do this, we simply run the following commands and confirm the installation:

```shell
dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
dnf install https://rpms.remirepo.net/enterprise/remi-release-8.rpm
dnf clean all
dnf makecache
```

1. the EPEL repo contains, as the name suggests, many *Extra Packages for Enterprise Linux* .
1. the REMI repo is **the** repository for PHP packages.
1. `dnf clean all` throws away the cache of `dnf` once.
1. `dnf makecache` will then rebuild it... can be done without `dnf clean all`, but *who cares* :-D 

## Swap modules

Now we exchange the modules once. For this we also use the `dnf` power :-)
> A short warning at this point, with mayor release changes problems can occur here, this should definitely be tested beforehand.
> Also, a backup is always a good idea that should be **implemented**!

> **Update:** I have carried out an update from PHP7.4 to PHP8.1. It went through without any problems.
> What are your experiences? Feel free to write it in the comments!

```shell
## Lists of available PHP modules
dnf module list php
## If a module is already activated, it must be reset beforehand.
dnf module reset php
## Select the new module
dnf module enable php:remi-8.0
```

## UPDAAAAATE!!!!!111elf

Finally, all that's missing is the update and we're done!

```shell
dnf update
```
