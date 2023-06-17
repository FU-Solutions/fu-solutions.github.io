---
title: MATRIX Calls - Host your own collaborationplatform
author: FarbzeN
date: 2021-11-17 14:35:00 +0100
categories: [Linux]
tags: [linux, ssh, config, shell]
math: true
mermaid: true
image:
  path: /assets/img/linux.png
  #lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Linux Logo
---

# MATRIX calls...
...from behind the __NAT__. Can you hear it?

## What (are you smoking)?
Haven't you heard? In the ocean of FOSS tools and VoIP clients that is out there, fighting for dominance, especially in times of COVID, it can be easy to lose the way and end up using proprietary cr... stuff.

Do not fret however! There is still hope and it comes in the form of one __matrix.org__!

## What (is MATRIX)?
__[MATRIX](https://matrix.org/)__ is a dedicated chat service that (potentially) provides everything from text-based conversations to VoIP-based conference calls where you and your colleagues watch the same [Michael Reeves PissBot](https://www.youtube.com/watch?v=tqsy9Wtr1qE) video at the same time.

### I said _potentially_
Why _potentially_? Well...

"Come closer..." I say as I look left and right, completely unsuspiciously, yet still fearing the wrath of the __Teams__ and __Slack__ gods that currently rule this disease-riddled mess of a planet.

"And the best thing? It's all end-to-end encrypted! Isn't that just amaz- THEY NOTICED US! RUN!"

## What (do I need to do)?
You can either use the centrally hosted service of matrix.org or, like a big boy, host a synapse server yourself.

For this, the process is pretty straight forward:
- Setup a linux server.
- Install and configure MATRIX synapse.
- Install and configure a TURN server.
- Reconfigure MATRIX synapse to use that TURN server.

## What (are we waiting for)?
That's the spirit!

### Linux server installation
First, you need a linux system with at least 1GB of Memory.
It also helps if the system is reachable via the following ports:

|Port|Protocol|Function|Comment|
| -------------- | -------------- | --------------| -------------- |
|443|TCP|HTTPS|Needed for general client functionality.|
|3478|UDP|TURN (VoIP)|Only neccessary if the TURN server is installed on the same Linux system. Can be ignored if the TURN traffic should be encrypted.|
|5349|TCP|TURNs (VoIP)|Only neccessary if the TURN server is installed on the same Linux system and the TURN traffic should be encrypted.|

Everything else is pretty much fair game and completely up to your use-case.

Feel free to experiment!

### MATRIX synapse installation
After the linux system is setup and ready, you can proceed with the installation of MATRIX synapse.

For simplicities sake, please follow the documentation provided by [__matrix.org__](https://www.matrix.org):

[MATRIX synapse Installation](https://matrix-org.github.io/synapse/latest/setup/installation.html)

Please also make sure you check out the following segments:
- [TLS certificates](https://matrix-org.github.io/synapse/latest/setup/installation.html#tls-certificates)
- [User registration](https://matrix-org.github.io/synapse/latest/setup/installation.html#registering-a-user)

Once you are done, you should be able to connect to your new __synapse__ server over a MATRIX client like [Element](https://element.io/get-started).

### TURN server installation
At this moment, there are two ways to establish voice and video calls with your new __synapse__ instance:

- Peer-to-Peer connection
  - This requires both clients to enable this setting in a client, that actually supports Peer-to-Peer VoIP calls. Also this makes your Public IP address visible to other clients.
- Let your __synapse__ server delegate calls to turn.matrix.org.
  - It should be obvious that this more or less defeats the purpose of a self-hosted MATRIX synapse instance.

To circumvent this, you have to set up a TURN server. This server has the specific job to handle calls for your MATRIX synapse instance so you don't have to worry about external factors.

This TURN server can be hosted on the same system as your __synapse__ server. Just make sure the two services can reach each other and the client can reach both of them.

Please follow the following documentation to set up your TURN server:

[TURN server installation](https://matrix-org.github.io/synapse/latest/turn-howto.html)

Once you restarted the TURN server, the __synapse__ server, and your client, you are able to place and receive calls over your new __synapse__ instance.

## The NAT part
One part that seems to be an ongoing issue with TURN(s) communication via NAT.

So, how did we solve this?

First, we made our Proxmox Host available via 5349 (TCP) and routed incoming traffic to the firewall. From there we routed the pakets over to our TURN/Synapse server:

```text
                     +--------+
                     |Internet|
                     +----+---+
                          |
                                                   +----
                        5349                    +--+ external ip on proxmox
                                                |  | nat via iptables to firewall
                          |                     |  +----
                          |                     |
    +---------------------+--------ProxMox-+    |
    |                     |                +----+
    |                +----v---+            |
    |           +----+Firewall|            |
    |           |    +--------+            |
    |           |                          |
    |     NAT 5349                         |
    |           |                          |
    |           |                          |
    |           |  +-------------------+   |
    |           +-->Synapse/TURN server|   |
    |              +-------------------+   |
    |                                      |
    |                                      |
    +--------------------------------------+
```

After we had all this in place, it was time for the all important test call:
<p align="center">
<img src="https://www.fu-solutions.de/wp-content/uploads/2021/11/synapse-turn-call-test-300x199.png" />
</p>
We are still working on that weird bug that makes me look like a low budget version of Chica from "Five Nights At Freddy's".

Other than that, we have yet to notice any problems with using this setup.

With that said, I hope you will have just as much success in setting up your MATRIX synapse instance! Good luck!
