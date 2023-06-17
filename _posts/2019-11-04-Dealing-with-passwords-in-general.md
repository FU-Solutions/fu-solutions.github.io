---
title: Dealing with passwords in general
author: w3ich3rt
date: 2021-09-02 14:35:00 +0100
categories: [Linux]
tags: [cluster, code, component, components, config, container, control, kubernetes, linux, monitoring, node node, parameter, parameters, plane, resource, resources, user, UserParameter, UserParameters, users, worker, zabbix]
math: true
mermaid: true
image:
  path: /assets/img/linux.png
  #lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Linux Logo
---


Passwords, a necessary evil and often the only protection against access to one's own identity (facebook) or other services. But now it happens that we administrators sometimes have to exchange passwords and usernames. Often also for several users. Further one must ask oneself then still the question, how one transfers the passwords and user names surely to the customer/user... simply via mail... no. Into an Excel... or via telephone?
All somehow not a cool option. Mail is pretty insecure because still not fully encrypted in the communication path, not to mention the content. Telephone... hmm, if every character of the highly secure password will be understood correctly? I don't know. Furthermore, as an admin you don't want to know the user's password, do you?

## Transfer of passwords

So, in general, the handling of passwords as an administrator is divided into two parts:
1. secure transmission of passwords to third parties
2. generation of passwords (without our knowledge of their content) according to a certain complexity.

Now there are various solutions for this. For the generation of passwords there are many tools. Under Linux there would be `pwgen` or `openssl` or with a little "playing" and "truncating" the use of `/dev/random/`. For individual cases I liked to use [`pwgen`](https://linux.die.net/man/1/pwgen). But that's not the point here - anyway, have a look ;).

For transferring passwords I decided to use OneTime-Links at some point and looked at different operators. Here are also some services that create such a link for sharing secrets. Without going into detail about the decision process now *(It's f**king open source)* I increasingly use [OneTimeSecret (OTS)](https://onetimesecret.com/) with growing enthusiasm.
+ Open Source
+ Via API Responsive
+ Selfhosting as well as hosted possible
+ Open Source

Here also the link to the [Github](https://github.com/onetimesecret/onetimesecret) of OTS.
So far so good - a secure transfer possibility as well as the tool to create passwords is found. But now it is not really nice to have a list of six users or just one user and for which two users have to be created and the passwords have to be distributed.

## Link generation

Therefore ~~(and because we are lazy)~~ we thought about how we can simplify the generation of passwords and one-time links. This resulted in the small but nice project "[ots-generate-password](https://github.com/FU-Solutions/ots-generate-password)" project.
This is a tool to generate passwords (in various ways) and then pass the links to users. We will of course continue to feed this tool with features.

## Tool ots-generate-password

The tool can be found in our Github under the link "[ots-generate-password](https://github.com/FU-Solutions/ots-generate-password)" and can be downloaded easily. It is a simple bashscript that uses API keys to access www.onetimesecret.com or the own solution of OTS and executes the corresponding steps via API requests. You can pass usernames, recipients as well as passwords and whole lists. When processing lists, the returns of the links sorted by usernames are also output in a CSV.
Here the small help output, for a feeling to the version 1.0.

```shell
	-W Use this script with power!
	-V More output as usual - for debugging.
	-L Chose the amount of characters in the password.
			Default: 20
	-v Version and Release of this script.
	-R Specify a recipient. Value needs to be a valid email address.
			The recipient will be set in ots.
	-U Specify a username. The username will also be set in OTS Secret.
	-f File with usernames and passwords. The username is mandatory.
				Please use the following format for the file:
				First column Username and second column password.
				Please use a TAB as delimeter.
				Please use no headerrow!
			If a file is passed to the script, the output will also be safed to a file.
	-O Specify an outputfile. 
			Default: 
	-P Specify a password, that will be send via ots link.
	-t Only check if the ots instance is fine and reachable.
```
