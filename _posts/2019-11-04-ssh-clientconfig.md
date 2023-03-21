---
title: SSH / Clientconfig – Unknown wrongfully
author: w3ich3rt
date: 2019-11-4 14:35:00 +0100
categories: [Linux]
tags: [linux, ssh, config, shell]
math: true
mermaid: true
image:
  path: /assets/img/linux.png
  #lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Linux Logo
---

# SSH / Clientconfig – Unknown wrongfully

If you have several Linux servers in your care and are allowed to maintain them, you surely know the problem of different SSH keys for different customers, environments or security aspects. Now of course there are several solutions, one approach could be to use the [putty agent](https://the.earth.li/~sgtatham/putty/latest/w64/pageant.exe) (under Windows). But if you work with a Linux client or use one in a private environment, it will be difficult. Of course there are also possible agents like keychain. But there are also possibilities with the already installed openssh-client, which I have to introduce to you here. For some reason this possibility is quite unknown, we want to change that. By the way this usage also works for rsync and other SSH based tools.

## Config file for the SSH client

With the configfile located in the .ssh directory of the userhome, you can store all possible settings for the SSH client, including connection settings like:

```shell
ForwardAgent yes 
ForwardX11 yes
```

But what I actually wanted to talk about is the fact that you can also pass the `IdentityFile` at this point, even bound to a host context. Let's assume you have one key for the host fu-solutions.de and another for weichert.it and don't want to pass usernames or the keyfile. Without setting up an alias. The corresponding host context in `~/.ssh/config` would look like this:

```shell
ForwardAgent yes 
ForwardX11 yes

Host fu-solutions.com 
    User Admin1 
    IdentityFile ~/.ssh/key-file1 
Host soft.it 
    User Admin2  
    IdentityFile ~/.ssh/key-file2
```

Now it is enough to enter the command line `ssh weichert.it` or `ssh fu-solutions.de` and after a confirmation the connection is automatically established with Admin2 or Admin1 (depending on the destination) and the corresponding `IdentityFile`. Via `Host` or `Match` the targets or environments can be differentiated.

```shell
# Used for connections to fu-solutions.de
Host fu-solutions.de
  User Admin1
  IdentityFile ~/.ssh/key-file1
# Used for connections to weichert.it
Host weichert.it
  User Admin2
  IdentityFile ~/.ssh/key-file2
# Used for all host with domain "domain.tld"
# without an identityfile
Host *.domain.tld
  User Admin3

# Match is used for combining host conditions.
```

This should be an incentive at this point, I hope I could give you a new and interesting opportunity to organize the SSH connections on a jumphost or your client. More information about the possibilities and possible configuration parameters can be found on the page [ssh.com](https://www.ssh.com/ssh/config/) or in the manpage for [ssh_config(5)](http://man.openbsd.org/OpenBSD-current/man5/ssh_config.5).
