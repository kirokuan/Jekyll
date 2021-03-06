---
layout: post
title: "Sql Server: find Bottleneck with sp_whoisactive"
date: 2016-05-19 00:04:06
tags: mssql sql sp_whoisactive
description: turing performance with sp_whoisactive 
---

I attended a sql meetup months ago, and the trainer introoduced a tool called [sp_whoisactive](https://www.brentozar.com/archive/2010/09/sql-server-dba-scripts-how-to-find-slow-sql-server-queries/)

I found it really powerful!! Since sp_who, sp_who is so simple and it takes more script to find out the problem.

Recent case is that I found a stored pocedure seemed to keep runnning in dashboard of activity monitor. It's kind of wired that the stored procedure is a initialized one,though it's heavy it only run once upon the application start.
{% highlight SQL %}
sp_whoisactive @get_plans = 1,@get_task_info = 2 
{% endhighlight %}

![screenshot]({{ site.baseurl | prepend:site.url}}/images/sql2.png){: .center-image }*Query Result*

 
After I ran this, it shows that the sp is running for 17 hr...It's so surprising, since the standard timeout for sp is 30 seconds.
and it's status is suspended, which means that it's waiting for somthing like IO, network...etc.

There is a field called "wait_info", it shows as "(8x: 64326765/64326765/64326766ms)CXPACKET:0, (1x: 1497ms)ASYNC_NETWORK_IO"
*Although without "@get_task_info = 2", there is wait_info as well, using "@get_task_info = 2" makes the field more informative.*

{% highlight SQL %}
SELECT  wt.session_id, 
    ot.task_state, 
    wt.wait_type, 
    wt.wait_duration_ms, 
    wt.blocking_session_id, 
    wt.resource_description, 
    es.[host_name], 
    es.[program_name] 
FROM  sys.dm_os_waiting_tasks  wt  
INNER  JOIN sys.dm_os_tasks ot ON ot.task_address = wt.waiting_task_address 
INNER JOIN sys.dm_exec_sessions es ON es.session_id = wt.session_id 
WHERE es.is_user_process =  1 
{% endhighlight %}

This can retrieve the wait info as well, and it shows that
![screenshot]({{ site.baseurl | prepend:site.url}}/images/sql.png){: .center-image }*Query Result*

So x8 means there are 8 CXPACKET events
I just looked up for this wait info for [CXPACKET](https://www.brentozar.com/archive/2013/08/what-is-the-cxpacket-wait-type-and-how-do-you-reduce-it/)

There is a great metaphor that query is to heavy, so it try to use mutithread to handle but eventually it cost much more.

In this case, finally I added a non-clustered index to table it query to make it run faster so that maybe it doesn't need to exchange so maany packets.

