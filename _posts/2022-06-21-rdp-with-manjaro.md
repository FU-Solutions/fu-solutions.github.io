---
title: XRDP with Manjaro (XFCE)
author: w3ich3rt
date: 2023-06-20 08:00:00 +0100
categories: [linux]
tags: [manjaro, linux, remote, rdp, secure, howto, xfce]
math: true
mermaid: true
image:
  path: /assets/img/manjaro.jpg
  #lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Modern logo with green light for Manjaro.
---

# Linux remote via RDP

I have a lot of desktop linux machines in my homelab. There is a `kali` for my security and CTF stuff, a `manjaro` for some tests and some other for testing the distros and desktop-manager. Because using them via a browser is not so much fun, I wanted to use them via RDP from my Windows Client.

After installing the XRDP tools, you can connect via the "Remote Desktop"-tool from Windows directly via Port 3389.

## Problems on Manjaro

While on Kali there is this [howto](https://www.kali.org/docs/general-use/xfce-with-rdp/), you can use to configure the machine and set up all things, on manjaro it is a littlebit more challenging.
I tried to install the packages and after starting the services I could connect, but after login the session terminated.
So I did some research and found, that you need to do some modifcations.

```shell
sudo pacman -Sy base-devel xorg-server-devel yay
yay -S xrdp xorgxrdp-git
echo "allowed_users = anybody" | sudo tee -a /etc/X11/Xwrapper.config
sed -i '12s/xfce-session/xfce4-session/g' ~/.xinitrc
sed -i '42s/ --exit-with-session//g' ~/.xinitrc
sed -i '66s/\$1/\$SESSION/g' ~/.xinitrc
sudo systemctl enable --now xrdp
```

- The first line installs the packages you’ll need to build and install xrdp and its friends.
- The second line installs xrdp and xorgxrdp-git, which are sort of the point of this exercise
- The third line permits you to enter an X (desktop) session without being at the computer – sorta important if your goal is to remote in.
- Lines 4-6 correct some typographical issues that prevent xrdp from running normally.
- Lines 7 enable and start the xrdp service.

So after doing this, everything should work as intended.

## Closing rdp connection - Linux mint

> Just a short hint!

I recently tried `xrdp` with Linux Mint 21.1 and the cinnamon desktop.
So I started the virtual machine, just installed `xrdp` with `apt` and changed the permissions to anybody. But after login in, my session always terminated after login.
The reason for that is, that you cannot run a `xrdp`-session while still logged-in on the virtual machine.
