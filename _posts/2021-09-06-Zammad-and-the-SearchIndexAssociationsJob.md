---
title: Zammad and the "SearchIndexAssociationsJob"
author: w3ich3rt
date: 2021-09-06 14:35:00 +0100
categories: [Linux]
tags: [linux, ssh, config, shell]
math: true
mermaid: true
image:
  path: /assets/img/linux.png
  #lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Linux Logo
---

> **Update:** Updated the command to rebuild the zammad searchindex.

I would like to briefly tell you about a small and quickly solvable problem that hit me in my daily work environment. Recently I updated a server including the Zammad<sup>1</sup> installation and prombt got some errors displayed.

## Error messages

```text
Failed to run background job #1 'SearchIndexJob' X time(s) with Y attempt(s).
Failed to run background job #2 'SearchIndexAssociationsJob' X time(s) with Y attempt(s).
```

Errors in the log files

```shell
Payload size: 0M
E, [2021-09-05T11:08:35.762780 #8395-46942827529260] ERROR -- : Retrying SearchIndexAssociationsJob in #<Proc:0x00005563744e6308@/opt/zammad/app/jobs/search_index_job.rb:6 (lambda)> seconds, due to a StandardEr
ror. The original exception was nil.
I, [2021-09-05T11:08:35.976602 #8395-46942827529260]  INFO -- : 2021-09-05T11:08:35+0200: [Worker(host:cs-zammad pid:8395)] Job SearchIndexAssociationsJob [36b545f8-a4fa-4016-8901-b009c0fbea7a] from DelayedJob(
default) with arguments: ["StatsStore", 2] (id=118379) (queue=default) COMPLETED after 0.2495
I, [2021-09-05T11:08:36.005992 #8395-46942827529260]  INFO -- : 2021-09-05T11:08:36+0200: [Worker(host:cs-zammad pid:8395)] Job SearchIndexAssociationsJob [a0db1f6f-5b0b-4a32-ba3c-c6e7511854b1] from DelayedJob(
default) with arguments: ["StatsStore", 3] (id=118380) (queue=default) RUNNING
E, [2021-09-05T11:08:36.054527 #8395-46942827529260] ERROR -- : Unable to process post request to elasticsearch URL 'http://localhost:9200/zammad_production_stats_store/_doc/3?pipeline=zammad85907522362'. Check
 the response and payload for detailed information: 

Response:
<UserAgent::Result:0x000055637528a4c8 @success=false, @body="{\"error\":{\"root_cause\":[{\"type\":\"illegal_argument_exception\",\"reason\":\"pipeline with id [zammad85907522362] does not exist\"}],\"type\":\
"illegal_argument_exception\",\"reason\":\"pipeline with id [zammad85907522362] does not exist\"},\"status\":400}", @data=nil, @code="400", @content_type=nil, @error="Client Error: #<Net::HTTPBadRequest 400 Bad
 Request readbody=true>!", @header={"x-elastic-product"=>"Elasticsearch", "warning"=>"299 Elasticsearch-7.14.1-66b55ebfa59c92c15db3f69a335d500018b3331e \"Elasticsearch built-in security features are not enabled
. Without authentication, your cluster could be accessible to anyone. See https://www.elastic.co/guide/en/elasticsearch/reference/7.14/security-minimal-setup.html to enable security.\"", "content-type"=>"applic
ation/json; charset=UTF-8", "content-length"=>"149"}>

Payload:
{"id":3,"stats_storable_type":"User","stats_storable_id":6,"key":"dashboard","data":{"StatsTicketReopen":{"used_for_average":0,"percent":0,"average_per_agent":0.0,"state":"supergood","count":0,"total":0},"Stats
TicketWaitingTime":{"handling_time":0,"average_per_agent":0,"state":"supergood","percent":0.0},"StatsTicketEscalation":{"used_for_average":0,"average_per_agent":0.0,"state":"supergood","own":0,"total":0},"Stats
TicketChannelDistribution":{"channels":{"email":{"icon":"email","inbound":0,"outbound":0,"inbound_in_percent":0,"outbound_in_percent":0},"phone":{"icon":"phone","inbound":0,"outbound":0,"inbound_in_percent":0,"
outbound_in_percent":0},"web":{"icon":"web","inbound":0,"outbound":0,"inbound_in_percent":0,"outbound_in_percent":0}}},"StatsTicketLoadMeasure":{"used_for_average":0.0,"average_per_agent":0.3,"percent":0.0,"sta
te":"superbad","own":0,"total":1,"average_per_agent_in_percent":0.0},"StatsTicketInProcess":{"used_for_average":0,"average_per_agent":0.0,"state":"supergood","in_process":0,"percent":0,"total":0}},"created_by_i
d":1,"created_at":"2021-04-14T12:25:28.637Z","updated_at":"2021-09-04T21:21:07.215Z","stats_storable":{"id":6,"organization_id":2,"login":"zabbix@cloudsupplies.de","firstname":"Zabbix","lastname":"Trigger","ema
il":"zabbix@cloudsupplies.de","image":null,"image_source":null,"web":"","phone":"","fax":"","mobile":"","department":"","street":"","zip":"","city":"","country":"","address":"","vip":false,"verified":false,"act
ive":true,"note":"","last_login":"2021-04-14T12:17:36.546Z","source":null,"login_failed":0,"out_of_office":false,"out_of_office_start_at":null,"out_of_office_end_at":null,"out_of_office_replacement_id":null,"pr
eferences":{"notification_config":{"matrix":{"create":{"criteria":{"owned_by_me":true,"owned_by_nobody":true,"subscribed":true,"no":false},"channel":{"email":true,"online":true}},"update":{"criteria":{"owned_by
_me":true,"owned_by_nobody":true,"subscribed":true,"no":false},"channel":{"email":true,"online":true}},"reminder_reached":{"criteria":{"owned_by_me":true,"owned_by_nobody":false,"subscribed":false,"no":false},"
channel":{"email":true,"online":true}},"escalation":{"criteria":{"owned_by_me":true,"owned_by_nobody":false,"subscribed":false,"no":false},"channel":{"email":true,"online":true}}}},"intro":true,"tickets_closed"
:549,"tickets_open":1},"updated_by_id":6,"created_by_id":3,"created_at":"2021-04-14T12:16:33.650Z","updated_at":"2021-09-04T22:01:14.609Z","fullname":"Zabbix Trigger"}}
```

These are mostly related to the underlying Elasticsearch installation.

## Possible causes

It can happen during updates that the plugin ingest-attachment is lost, this must then be reinstalled.
Further it can be that the search index of the Elasticsearch database must be rebuilt, this was the case with me.

## Solution

In my case I was able to get the server running again by rebuilding the search index. You can also do this safely with the following command.
If your Zammad installation is very busy, you should stop the Zammad service first!

```shell
## In case your installation is under heavy load
user@system:~/ sudo systemctl stop zammad.service
user@system:~/ sudo zammad run rake zammad:searchindex:rebuild
## After rebuild process finished and if you stopped your instance start zammad
user@system:~/ sudo systemctl start zammad.service
```

The process is relatively conversational and should generate you something like the following outputs:

```shell
drop indexes...done
delete pipeline (pipeline)...done
create indexes...done
create pipeline (pipeline)... done
reload data...
 reload Cti::Log
  - started at 2021-09-05 09:10:10 UTC
  - took 0 seconds
 reload StatsStore
  - started at 2021-09-05 09:10:10 UTC
        4/4
  - took 2 seconds
 reload User
  - started at 2021-09-05 09:10:13 UTC
        5/5
  - took 2 seconds
 reload Organization
  - started at 2021-09-05 09:10:15 UTC
        1/1
  - took 0 seconds
 reload Chat::Session
  - started at 2021-09-05 09:10:16 UTC
  - took 0 seconds
 reload ticket
  - started at 2021-09-05 09:10:16 UTC
        100/553
        200/553
        300/553
        400/553
        500/553
        553/553
  - took 103 seconds
 reload KnowledgeBase::Answer::Translation
  - started at 2021-09-05 09:12:00 UTC
  - took 0 seconds
 reload KnowledgeBase::Category::Translation
  - started at 2021-09-05 09:12:00 UTC
  - took 0 seconds
 reload KnowledgeBase::Translation
  - started at 2021-09-05 09:12:00 UTC
  - took 0 seconds
 reload Ticket::State
  - started at 2021-09-05 09:12:00 UTC
        7/7
  - took 3 seconds
 reload Ticket::Priority
  - started at 2021-09-05 09:12:03 UTC
        3/3
  - took 0 seconds
 reload Group
  - started at 2021-09-05 09:12:03 UTC
        1/1
  - took 0 seconds
```

After that, the monitoring status should be green again.

[![Zammadstatusimage](https://www.fu-solutions.de/wp-content/uploads/2021/09/zammad_green.png "Zammadstatusimage")](https://www.fu-solutions.de/wp-content/uploads/2021/09/zammad_green.png "Zammadstatusimage")

1: Zammad is an opensource ticket system and you can find it at [Zammad.com](https://zammad.com/de "Zammad.com")