---
title: DNS Timeouts in DUALSTACK Environments
author: w3ich3rt
date: 2016-10-3 14:35:00 +0100
categories: [Linux]
tags: [dns, dualstack, linux, oracle, shell, timeout]
math: true
mermaid: true
image:
  path: /assets/img/linux.png
  #lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Just a logo from fu-solutions
---

A few days ago, we had some strengh phenomenon on our Oracle RAC Servern with TNSListener Timeouts.

```shell
EM Event: Critical:LISTENER_SCAN3_rac-scan - Listener response to a TNS ping is 5,010 msecs
```

After some research and many hours of analysing, we could get the right information. The server wants to get the A-Record and the AAAA-Record (quadA-Record) after sending his dns-request. That you can see in this TCPDUMP.

```shell
2016-10-13 15:22:54.247889 IP (tos 0x0, ttl 64, id 11169, offset 0, flags [DF], proto UDP (17), length 75)
    11.20.22.101.23397 &gt; 11.20.41.10.domain: [bad udp cksum 5c97!] 7836+ A? rac-scan.domain.local. (47)
2016-10-13 15:22:54.247895 IP (tos 0x0, ttl 64, id 11170, offset 0, flags [DF], proto UDP (17), length 75)
    11.20.22.101.23397 &gt; 11.20.41.10.domain: [bad udp cksum 6ac8!] 53901+ AAAA? rac-scan.domain.local. (47)
2016-10-13 15:22:54.248504 IP (tos 0x0, ttl 127, id 30819, offset 0, flags [none], proto UDP (17), length 123)
    11.20.41.10.domain &gt; 11.20.22.101.23397: [udp sum ok] 7836* q: A? rac-scan.domain.local. 3/0/0 rac-scan.domain.local. [1h] A 11.20.22.104, rac-scan.domain.local. [1h] A 10.60.22.105, rac-scan.domain.local. [1h] A 11.20.22.103 (95)
2016-10-13 15:22:59.251573 IP (tos 0x0, ttl 64, id 11171, offset 0, flags [DF], proto UDP (17), length 75)
    11.20.22.101.23397 &gt; 11.20.41.10.domain: [bad udp cksum 5c97!] 7836+ A? rac-scan.domain.local. (47)
```

Okay this behavior is the normal way in dualstack enviremonts. The problem is, that some operationsystems and some hardware send the request over the same socket, but close the socket after the first response. Because of that, the second request waits and waits and waits.... (till Timeout).
So... you can deactivate the IPv6 if you dont need that, or you could set the following option for the resolver.

```shell
man resolv.conf
[... abridged output ...]
...
single-request (since glibc 2.10)
     sets RES_SNGLKUP in _res.options.  By default, glibc performs IPv4 and IPv6 lookups in parallel since version 2.9.  Some appliance DNS servers cannot handle these queries properly and make the
    requests time out.  This option disables the behavior and makes glibc perform the IPv6 and IPv4 requests sequentially (at the cost of some slowdown of the resolving process).
single-request-reopen (since glibc 2.9)
    The  resolver uses the same socket for the A and AAAA requests.  Some hardware mistakenly only sends back one reply.  When that happens the client sytem will sit and wait for the second reply.
    Turning this option on changes this behavior so that if two requests from the same port are not handled correctly it will close the  socket and  open  a  new  one  before  sending  the  second
    request.
...
```

We decided to take option two and this is how the resolv.conf is looking now.

```shell
[root@rac01 ~]# cat /etc/resolv.conf
options single-request-reopen
search domain.local
nameserver 11.20.41.10
nameserver 11.20.41.11
```

Now the machines running like charme ;)
