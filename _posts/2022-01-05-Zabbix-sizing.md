---
title: Zabbix Sizing
author: w3ich3rt
date: 2022-01-05 14:35:00 +0100
categories: [Linux]
tags: [linux, ssh, config, shell]
math: true
mermaid: true
image:
  path: /assets/img/zabbix.png
  #lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Zabbix Logo
---

<span style="color:red">**Article updated on: 05.01.2022**</span>

Zabbix, a very comprehensive monitoring system, which is a very nice alternative to the well-known monitoring systems [Cacti](http://www.cacti.net/ "Cacti") and [Nagios](https://www.nagios.org/ "IT"). In addition to extensive existing templates, it also offers a very high level of customisability and a strong community.
But at the beginning of every monitoring system there is first of all the planning for the structure (architecture) and of course the sizing of the actual hardware resource. The requirements are of course provided directly by Zabbix and can be found here: [Zabbix Requirements](https://www.zabbix.com/documentation/current/en/manual/installation/requirements "Zabbix").

Another interesting aspect in this case is of course the sizing of the database that holds the settings and historical data of the Zabbix installation. In the company where I have worked so far and am currently allowed to work, we use Zabbix for pretty much any kind of monitoring and thus quite a lot of data comes together.
In order to determine the database sizing as simply and quickly as possible, I have developed a Zabbix-DB-Calculator in this context. This calculates the required database size directly, depending on the requirements.
Click here for the [Zabbix DB Calculator](https://code.cloudsupplies.de/cloudsupplies-services/ZabbixCalc). It is available as a Python Flask application and as a Wordpress plugin and is widely customisable.
Of course you can also get directly to the calculator via our [Blog](https://www.fu-solutions.de/zabbix-calculator/).
