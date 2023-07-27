---
title: Howto - Run a Flaskapp as a service with systemd
author: w3ich3rt
date: 2023-06-19 20:00:00 +0100
categories: [Linux]
tags: [linux, howto, flask, python, systemd, flaskapp, service]
math: true
mermaid: true
image:
  path: /assets/img/linux.png
---

# General

I recently wrote myself a flaskapp using python. The content is not so important, but I would like to run the thing permanently. Preferably controlled by the operating system. So I can then, for example with a reverse proxy like `nginx`, access the service and do not have to worry after a restart that I forget something.

So how do I bind my Flaskapp with `systemd`? I simply create my own `unit` under `/etc/systemd/system` with the following content:

`vi /etc/systemd/system/flaskapp.service`

```shell
[Unit]
Description=My Flask App
After=network.target

[Service]
User=<username>
WorkingDirectory=<path to your app directory>
ExecStart=<app start command>
Restart=always

[Install]
WantedBy=multi-user.target
```

Now I still need to add the new service. I can do this via reboot of the system or simply via `systemctl daemon-reload`. This will refresh the `systemd` configuration and the new service will be recognized.

To make the autostart of the service work, I have to issue the following commands:

```shell
# This will enable the start on boot and start the service directly
systemctl enable --now flaskapp.service
```

Of course, you can also do this separately:

```shell
systemctl enable flaskapp.service
systemctl start flaskapp.service
```

