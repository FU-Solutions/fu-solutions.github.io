---
title: KASM Workspaces - Install on rocky linux
author: w3ich3rt
date: 2022-02-16 14:35:00 +0100
categories: [Linux]
tags: [linux, ssh, config, shell]
math: true
mermaid: true
image:
  path: /assets/img/linux.png
  #lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Linux Logo
---

In a video from [networkchuck](https://www.youtube.com/channel/UC9x0AN7BWHpCDHSm9NiJFJQ "networkchuck"), he introduced [kasm workspaces](https://www.kasmweb.com/ "kasm workspace"). A tool or more like a webservice you can host (wherever you want) to start encapsulated (kind of more secure) desktop environments or apps in a docker container, that will be streamed to your browser. Because [Qube OS](https://www.qubes-os.org/ "Qube OS") is not compatible with my hardware, I instandly thought, oh great! I need this RIGHT NOW.
Recently I setup an QEMU/KVM environment on my laptop, so I just needed to create a new virtual machine and fire this thing up - lets do this.
Before reading the install docs on the [kasm website](https://www.kasmweb.com/ "kasm website"), I created a new virtual machine and installed rocky linux, cause I wanna give it a try. Okay it is not supported, but centos is, so it will propably work.

## Install kasm workspace docker

After rocky os installation process finished and I got a prombt, I followed the steps in the documentation.

In short:

#### Create swap

```shell
sudo dd if=/dev/zero bs=1M count=1024 of=/mnt/1GiB.swap
sudo chmod 600 /mnt/1GiB.swap
sudo mkswap /mnt/1GiB.swap
sudo swapon /mnt/1GiB.swap
```

#### Verify swap file exists

```
cat /proc/swaps
```

To make the swap file available on boot

```shell
echo '/mnt/1GiB.swap swap swap defaults 0 0' | sudo tee -a /etc/fstab
```

## Download kasm worskpace
You can download the archive from the [kasm download page](https://www.kasmweb.com/downloads.html "kasm download page").... suprise.
After downloading, you need to unpack the archive and run the install script. It will run an install_dependencies script, before installing all needed files and docker images.

```shell
tar -xf kasm_release*.tar.gz
sudo bash kasm_release/install.sh
```

> This script does not work with every distro! 

I needed to add the following lines in the `install_dependencies.sh` script to get the compatibility check succsessfully done and so the script is able to install all needed dependencies.

```shell

## Copy these lines and paste them directly after it

if [ "${OS_ID}" == '"centos"' ] && ( [ "${OS_VERSION_ID}" == '"7"' ] || [ "${OS_VERSION_ID}" == '"8"' ] ) ; then
    SUPPORTED='true'
    if [ $INSTALL_DOCKER == 'true' ] ; then
        install_centos ${OS_VERSION_ID}
    fi

    if [ $INSTALL_COMPOSE == 'true' ] ; then
        install_docker_compose
    fi
fi

## Change the "if" for OS_ID check and OS_VERSION_ID to rocky and the version your have installed.

if [ "${OS_ID}" == '"rocky"' ] && ( [ "${OS_VERSION_ID}" == '"7"' ] || [ "${OS_VERSION_ID}" == '"8.5"' ] ) ; then
    SUPPORTED='true'
    if [ $INSTALL_DOCKER == 'true' ] ; then
        install_centos ${OS_VERSION_ID}
    fi

    if [ $INSTALL_COMPOSE == 'true' ] ; then
        install_docker_compose
    fi
fi
```

Now the script will run through and download / installs all you need!