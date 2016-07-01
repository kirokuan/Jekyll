---
layout: post
title: "Using Kibana to Represent Geo info"
date: 2016-06-19 00:04:06
tags: Kibana GeoIp
description: Use Kibana, Logstash to analyse log and build a Geo ditribution 
---

I tried to use ELK( ElasticSearch, Logstash and Kibana) to analyse the log and pushed them to Kibana and build a representation.

Most steps are listed in the some tutorials like [this](https://www.digitalocean.com/community/tutorials/how-to-map-user-location-with-geoip-and-elk-elasticsearch-logstash-and-kibana)
and I aimed at analysing the iis log.

I downloaded are

* ElasticSearch 2.2
* Logstash 5.0.0 alpha (not sure why the version jumped to 5???)
* Kibana 4.5.1 Window

![screenshot]({{ site.baseurl | prepend:site.url}}/images/GeoIp.png){: .center-image }*GeoIp with User Distrubution*

Although I followed the tutorial and copy log parser for iis, but it didn't work. Using [grok debuuger](https://grokdebug.herokuapp.com/) may help.

Eventaully my analysis pattern look like

    grok {
		match => ["message", "%{TIMESTAMP_ISO8601:timestamp} %{WORD:filename} %{NOTSPACE:ComputerName} %{IPORHOST:hostip} %{WORD:method} %{URIPATH:page} %{NOTSPACE:query} %{NUMBER:port} %{NOTSPACE:username} %{IPORHOST:clientip} %{NOTSPACE:httpver} %{NOTSPACE:useragent} %{NOTSPACE:Cookie} %{NOTSPACE:Referer} %{NOTSPACE:Host} %{NUMBER:http_response} %{NUMBER:sub_response} %{NUMBER:sc_status} %{NUMBER:time_taken}"]
	}
   
And add GeoIp database to convert ip to geo position.
    geoip {
        source => "clientip"
        target => "geoip"
        database => "X:/GeoLite2-City.mmdb"
        add_field => [ "[geoip][coordinates]", "%{[geoip][longitude]}" ]
        add_field => [ "[geoip][coordinates]", "%{[geoip][latitude]}"  ]
    }
Some tutorial used GeoLiteCity.dat, but GeoLiteCity.dat didn't work for my logstash, which always threw exception of wrong format.
And I downloaded GeoLite2-City.mmdb from [their website](https://dev.maxmind.com/geoip/geoip2/geolite2/) 
And for the data that need to be converted as number, adding them in *mutate* part

    mutate {
	    remove_field => [ "log_timestamp"]
        convert => [ "[geoip][coordinates]", "float"]
        convert => ["bytesSent", "integer"]
        convert => ["bytesReceived", "integer"]
        convert => ["time_taken", "integer"]
    }
    
and configure the template for index to convert the data with properly typing.