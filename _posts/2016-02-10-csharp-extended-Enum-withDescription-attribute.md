---
layout: post
title: "Extend Enum to use Description Attribute"
date: 2016-02-10 23:04:06
tags: C# Enum
description: First Post to celebrate the blog done
---

Define a Attribute with string property
{% highlight csharp %}
    public class DescriptionAttribute : Attribute
    {
        public string Description { get; set; }

        public DescriptionAttribute(string desp)
        {
            Description = desp;
        }
    }
{% endhighlight %}

Extend the method of enum, so that all enum can use method GetDescription
{% highlight csharp %}
    public static class ExtendEnum
    {
        public static string GetDescription(this Enum e)
        {
            if (e == null) return null;
            var memInfo = e.GetType().GetMember(e.ToString());
            var attributes = memInfo[0].GetCustomAttributes(typeof(DescriptionAttribute), false).Where(a => a is DescriptionAttribute).Cast<DescriptionAttribute>().ToList();
            if (attributes.Any())
                return attributes[0].Description;
            return e.ToString();
        }
        public static T ParseFromDescription<T>(this Enum e, string description)
        {
            var typeT = typeof(T);
            var enumList = Enum.GetValues(typeT).Cast<T>();
            foreach (var x in from x in enumList
                              let memInfo = typeT.GetMember(x.ToString())
                              from m in memInfo
                              let attributes = memInfo[0].GetCustomAttributes(typeof		(DescriptionAttribute), false)
                .Where(a => a is DescriptionAttribute)
                .Cast<DescriptionAttribute>()
                .ToList()
                              where attributes.Any() && attributes[0].Description == description
                              select x)
            {
                return x;
            }
            return (T)
                Enum.Parse(typeT, description, true);
        }
    }
{% endhighlight %}

