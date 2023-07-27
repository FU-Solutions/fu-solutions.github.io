---
title: Howto - Analyze haproxy logs with ELK
author: w3ich3rt
date: 2023-07-26 20:00:00 +0100
categories: [linux]
tags: [elk, elasticsearch, kibana, logstash, loganalyzing, linux, howto, grokpattern]
math: true
mermaid: true
image:
  path: /assets/img/elkstack.svg
---

# General

We all know the situation in our everyday life, a customer calls and has a problem and does not know exactly what the problem is. To get to the bottom of the matter, you now have to check a buttload of log files or their contents. I'll put it this way, there are tasks in IT that are more fun.
I had such a case again and had no desire to look at everything by hand. Luckily I have an ELK stack running and thought I'd just throw the logs in there. Afterwards they are prepared and much better filterable.

## Import of the logs

Often the ELK stack can also parse these logs quite well, i.e. recognize the patterns within the logs and process them accordingly. In my case, unfortunately, this was not so easy to do. The `haproxy.log` was simply not recognized cleanly. Therefore I had to specify the "grok" pattern when importing the log. For the commercial `haproxy.log`s the pattern looks like this.

> Editor's note: The proxy logs are set to info and the pattern is only applied to the first 4 log entries.

```text
%{SYSLOGTIMESTAMP:timestamp} %{DATA:hostname} %{DATA:SyslogFacility}\[%{DATA:Haproxy_process}\]: %{DATA:client_ip}:%{INT:client_port} \[%{HAPROXYDATE:accept_date}\] %{NOTSPACE:frontend_name} %{NOTSPACE:backend_name}/%{NOTSPACE:server_name} %{INT:Tq}/%{INT:Tw}/%{INT:Tc}/%{INT:Tr}/%{INT:Tt} %{INT:http_status_code} %{NOTSPACE:bytes_read} - - ---- %{INT:actconn}/%{INT:feconn}/%{INT:beconn}/%{INT:srvconn}/%{NOTSPACE:retries} %{INT:srv_queue}/%{INT:backend_queue} "%{WORD:Method} %{URIPATHPARAM:request} HTTP/%{NUMBER:http_version}"
```

## Add GeoIP Processor:

During log analyses, it can still be quite nice to get additional information about the log entries. For example: Where do the accesses come from approximately? So I added a data processor to get the location of the IP address. We do this via a 'GeoIP' processor. 

```text
    {
      "geoip": {
        "field": "clientip"
      }
    }
```

Now we can import the log file and have access to a filterable view of the logs with various analysis options as well as graphical representations and of course the possibility to display the access locations on a world map.

I hope this little article helps you. If necessary, I will add a few more pictures later.
