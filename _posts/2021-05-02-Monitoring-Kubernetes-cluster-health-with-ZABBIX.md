---
title: Monitoring Kubernetes cluster health with ZABBIX
author: FarbzeN
date: 2021-05-02 14:35:00 +0100
categories: [Linux]
tags: [cluster, code, component, components, config, container, control, kubernetes, linux, monitoring, node node, parameter, parameters, plane, resource, resources, user, UserParameter, UserParameters, users, worker, zabbix]
math: true
mermaid: true
image:
  path: /assets/img/zabbix-kubernetes.png
  #lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Zabbix und Kubernetes Logo
---

Ever wanted to use ZABBIX to monitor Kubernetes resources?

Just recently, we got the chance to play around with some Kubernetes resources and create solutions to integrate them into existing environments.
One of the most popular topics that popped up was, of course, monitoring.

So we went to work and figured out a way to implement ZABBIX checks for monitoring Kubernetes clusters.

### Ingredients
To create a working ZABBIX monitoring solution for Kubernetes clusters and resources you need:
- 1 ZABBIX server/proxy installation
- 1+ Kubernetes cluster
  - 1 service account in the "kube-system" namespace
- 1 ZABBIX agent installed on a kubernetes node
  - 1 set of UserParameters
  - 1 kubeconfig placed in the ZABBIX agent directory

### Implementation
Got everything? Good! Now you can go to our brand new GitHub project and check out what to do next:

[![ZABBIX Kubernetes checks on GitHub](https://www.fu-solutions.de/wp-content/uploads/2021/05/zabbix_kubernetes_checks-1.png)](https://github.com/FU-Solutions/zabbix_kubernetes_checks)

