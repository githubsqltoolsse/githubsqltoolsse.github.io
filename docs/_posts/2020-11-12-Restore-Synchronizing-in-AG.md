---
layout: posts
title:  "Restore synchronizing in AG"
date:   2020-11-11 10:31:16 +0100
categories: TSql
tags: TSql Disaster-Recovery
---

Just a short note. When a secondary database stops synchronizing. This simple command can help. Eventually you must also reset the endpoint. I don't got the command at hand right now.

{% highlight sql %}

ALTER DATABASE MyDatabase SET HADR RESUME

{% endhighlight %}
