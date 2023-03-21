---
title: iperf – a great bandwidth measure tool
author: w3ich3rt
date: 2016-12-09 13:00:00 +0100
categories: [Linux]
tags: [analyse, iperf, linux, measure, perfomance, shell, tool]
math: truef
mermaid: true
image:
  path: /assets/img/linux.png
  #lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Zabbix Logo  
---

Today i want to introduce a little bandwith measure tool to you. `iperf`is a tool to measure the bandwith between to systems. It is a client/server application, so you need to install it on both sides. To install this little guy, i use the standard repository from the system iam working on. On a RHEL you need to add the [epel-repo](https://fedoraproject.org/wiki/EPEL) to the machines. On ubuntu or debian you would probably use

```shell
sudo apt-get install iperf
```

After you have installed it on both sides, you have to start the server on one side. You could do it like that

```shell
iperf -s
# if you want to use another port
iperf -s -p 12345
```

Now you should see something like that (i used the standardport in this example)

```shell
[root@weichert ~] # iperf -s
------------------------------------------------------------
Server listening on TCP port 5001
TCP window size: 85.3 KByte (default)
------------------------------------------------------------
```

On the other side you connect with iperf to the server and start the test. With the parameter `-r` you start a bidirectional test. With `-P 5` you will open 5 parallel client threads. The parameter `-w` will change the tcp window size.
Okay let's have a look on a simple test with iperf.

```shell
# Start the test on clientside
[root@test-vm ~] # iperf -c weichert.it -P 5
------------------------------------------------------------
Client connecting to weichert.it, TCP port 5001
TCP window size: 2.50 MByte (default)
------------------------------------------------------------
[  7] local 85.114.145.4 port 36718 connected with 85.114.145.4 port 12345
[  3] local 85.114.145.4 port 36714 connected with 85.114.145.4 port 12345
[  4] local 85.114.145.4 port 36715 connected with 85.114.145.4 port 12345
[  5] local 85.114.145.4 port 36716 connected with 85.114.145.4 port 12345
[  6] local 85.114.145.4 port 36717 connected with 85.114.145.4 port 12345
[ ID] Interval       Transfer     Bandwidth
[  3]  0.0-10.0 sec  6.19 GBytes  5.34 Gbits/sec
[  4]  0.0-10.0 sec  4.62 GBytes  3.98 Gbits/sec
[  5]  0.0-10.0 sec  4.76 GBytes  4.11 Gbits/sec
[  6]  0.0-10.0 sec  5.83 GBytes  5.03 Gbits/sec
[  7]  0.0-10.0 sec  6.35 GBytes  5.46 Gbits/sec
[SUM]  0.0-10.0 sec  27.8 GBytes  23.8 Gbits/sec
# The server generate some similiar output
[root@weichert ~] # iperf -s
------------------------------------------------------------
Server listening on TCP port 5001
TCP window size: 85.3 KByte (default)
------------------------------------------------------------
[  4] local 85.114.145.4 port 12345 connected with 85.114.145.4 port 36714
[  5] local 85.114.145.4 port 12345 connected with 85.114.145.4 port 36715
[  6] local 85.114.145.4 port 12345 connected with 85.114.145.4 port 36716
[  7] local 85.114.145.4 port 12345 connected with 85.114.145.4 port 36717
[  8] local 85.114.145.4 port 12345 connected with 85.114.145.4 port 36718
[ ID] Interval       Transfer     Bandwidth
[  4]  0.0-10.0 sec  6.19 GBytes  5.31 Gbits/sec
[  5]  0.0-10.0 sec  4.62 GBytes  3.96 Gbits/sec
[  7]  0.0-10.0 sec  5.83 GBytes  5.01 Gbits/sec
[  6]  0.0-10.0 sec  4.76 GBytes  4.09 Gbits/sec
[  8]  0.0-10.0 sec  6.35 GBytes  5.43 Gbits/sec
[SUM]  0.0-10.0 sec  27.8 GBytes  23.7 Gbits/sec
```

So far so good, but what if there is no iperf installed, or you cannot install it because of other restrictions? [Take a look to dd](https://www.weichert.it/en/test-your-bandwidth-with-dd-orand-netcat/). I wrote an short article to test bandwith with DD and/or Netcat.
