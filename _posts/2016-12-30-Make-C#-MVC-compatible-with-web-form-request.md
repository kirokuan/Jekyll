---
layout: post
title: "Make MVC web api compatible with www-url-encoded header"
date: 2016-12-30 00:00:00
tags: C# MVC web-api header
description: make MVC api support both customized application/x-www-form-urlencoded or application/JSON
---

Recently, we migrated the old projects to new MVC web api. we preserved the the api that is implemented by web-form. There is no problem for api, since the router can redirect the erequest to them. However, for new api,we need to make is compatible with form-encoded format.

Since our response is JSON-formatted, so we use JsonMediaTypeFormatter. Although there is a formatter called "FormUrlEncodedMediaTypeFormatter" to support x-www-form-urlencoded format, it doessn't work for our case. Because we used our own format, rather than formal one.

our request body is like obj={ A:"xx", B:"yyy"}, and the string is url encoded. 

According to the architeture of MVC api, I chose to implement a convetor with *DelegatingHandler*, so that it can convert the request before the route is determined.

{% highlight csharp %}

public class CustomizedHttpHandler : DelegatingHandler
{
        public readonly string JsonHeader = "application/json";

        protected override async Task<HttpResponseMessage> SendAsync(
            HttpRequestMessage request, CancellationToken cancellationToken)
        {
            if ((request.Content.Headers.ContentType != null &&
                 request.Content.Headers.ContentType.MediaType == JsonHeader))
                return await base.SendAsync(request, cancellationToken);
            var body = await request.Content.ReadAsStringAsync();
            var p = "";
            var param = HttpUtility.ParseQueryString(body);
            p = param["obj"];
            
            request.Content = new StringContent(p, Encoding.UTF8,
                    JsonHeader);
            return await base.SendAsync(request, cancellationToken);
        }

}

{% endhighlight %}

So we use the request header to judge if it use new or old format. 

Futher, we also use RouteAttribute to make api look like .aspx.

{% highlight csharp %}

        [AcceptVerbs("POST")]
        [Route("xyz.aspx")]
        public string test(object request)
        
{% endhighlight %}

