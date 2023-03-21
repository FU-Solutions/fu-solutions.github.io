---
title: Test your bandwidth with dd or/and netcat
author: w3ich3rt
date: 2016-12-09 13:00:00 +0100
categories: [Linux]
tags: [dd, iperf, linux, measure, netcat, perfomance, shell, ssh]
math: truef
mermaid: true
image:
  path: /assets/img/linux.png
  #lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Zabbix Logo  
---

If you dont have a iperf on your server and you cannot install it for some reasons my [iperf article](https://www.weichert.it/en/iperf-a-great-bandwidth-measure-tool/) will not help you to determine your throuphput. But you can check your bandwidth with some other tools. Most of them are standardtools on a server.
**Alternative one: netcat** With `netcat` you can do something similiar. First you need to open a listening port on one side, then you connect from another one.

```shell
# Serverside
[root@weichert ~] # netcat -vvnlp 12345 > /dev/null
listening on [any] 12345 ...
connect to [127.0.0.1] from (UNKNOWN) [127.0.0.1] 38340
sent 0, rcvd 149621964
# Clientside
time yes|netcat -v localhost 12345 &gt; /dev/null
localhost [127.0.0.1] 12345 (?) open
real    0m41.912s
user    0m40.796s
sys     0m7.252s
```

Now you have to do some math, yeah i know - we all like that. Multiply the bytes rcvd by 8 to get total bits, then divide by the time: Result is 28559 MBit/s => 28Gbit/s
**Alternative two: dd over ssh or netcat** Lets have a look to dd. Here you will create a transfer over `netcat` oder with `ssh` to the target.

```shell
dd if=/dev/zero bs=1024 count=512 | netcat -v localhost 12345
# you will get something like
536870912 bytes (537 MB) copied, 3.42426 s, 182 MB/s
# with ssh
dd if=/dev/zero bs=1024 count=512 | ssh user@testhost.de 'cat /dev/null'
536870912 bytes (537 MB) copied, 3.42426 s, 182 MB/s
```

You have to multiply the 182MB/s by 8 to get Mbit/s
Have fun with playing around with that.
