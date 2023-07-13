---
title: Howto - Install openstack on a virtual machine
author: w3ich3rt
date: 2023-06-27 18:00:00 +0100
categories: [linux]
tags: [openstack, linux, howto, container, iaas]
math: true
mermaid: true
image:
  path: /assets/img/openstack.png
  #lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Openstack Logo
---

# Virtual machine

The very first thing I did was set up a virtual machine in my Proxmox environment on Debian 11 Linux. I gave it 4 cores, 8 GB of memory and 100 GB of disk space. I also added two network cards. The one in the normal network, for access, and the second network card for an internal network. I'll spare myself the rollout process of a virtual machine ;-). 
Das Rocky Linux wurde in der Minimalkonfiguration inklusive Gast-Tools ausgerollt.

# Installation Openstack

Installing openstack on a node via [devstack](https://opendev.org/openstack/devstack) is as easy as making a slice of toast with cheese.

> A note at this point, normally you will not install Openstack with its different modules on a machine, that is really only for evaluation and some testing in this case. Normally you will roll out the different modules in a k8s[^1] environment.

We need an extra user with `sudo` permissions for the preparation and we simply create this via

```shell
sudo useradd -s /bin/bash -d /opt/stack -m stack
echo "stack ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/stack
sudo -u stack -i
```

Now we can clone the devstack repo and create a configuration file for installing Openstack.

```shell
git clone https://opendev.org/openstack/devstack
cd devstack
```

In the devstack directory we now create `local.conf` with the following content

```shell
cat << EOF >> devstack/local.conf
[[local|localrc]]
ADMIN_PASSWORD=MySecretPassword123
DATABASE_PASSWORD=\$ADMIN_PASSWORD
RABBIT_PASSWORD=\$ADMIN_PASSWORD
SERVICE_PASSWORD=\$ADMIN_PASSWORD
EOF
```

> Please never use such a password. This is for demonstration purposes only. Every password should be unique!!!1111

Now we can start the installation process with `stack.sh` by just typing 

```shell
./stack.sh
```

If there are no problems, the installation can take a long time, for me it was about 19 minutes.
But you are provided with enough passing shell output the whole time, so that no boredom arises.

Now you should be able to openstack via horizon using the link `http://<YOUR IP>/dashboard`.

## Problems

### Permissions

I still had a problem with some permissions of the `/opt/stack` directory during the installation, which I had to correct manually.


```shell
cd /opt/
chmod 755 stack
```

### Repository

Under my Debian installation I still had the problem that the ONV software could not be installed cleanly. Here it was pointed out that the packages are too old and were replaced.
I had to add the backports in the repo so that I could install on the appropriate packages (which are required by devstack).

Just edit the `/etc/apt/source.list` and add this line:

```text
deb http://deb.debian.org/debian bullseye-backports main
```

### Closing rdp connection

> Just a short hint!

I recently tried `xrdp` with Linux Mint 21.1 and the cinnamon desktop.
So I started the virtual machine, just installed `xrdp` with `apt` and changed the permissions to anybody. But after login in, my session always terminated after login.
The reason for that is, that you cannot run a `xrdp`-session while still logged-in on the virtual machine.

----

[^1]: Fancy name for kubernetes, just if you don't know :-)
