---
layout: post
title: Apache rewrite rule to replace %20-%20 with a dash (#) in URLs
date: '2020-02-19T08:02:00.001+01:00'
author: Torben Hansen
tags:
- "%20-%20"
- rewrite
- apache
- dash
- Excel
modified_time: '2020-02-19T08:02:36.620+01:00'
blogger_id: tag:blogger.com,1999:blog-6517038209122183182.post-1709027057669901538
blogger_orig_url: http://www.derhansen.de/2020/02/apache-rewrite-rule-to-replace-20-20.html
permalink: /2020/02/apache-rewrite-rule-to-replace-20-20.html
---

Some old(?) versions of Microsoft Excel replace a dash (**#**) in an URL with "**%20-%20**". The following example
shows, how Excel transforms URLs:

**Original URL:**

https://www.domain.tld/some/path/#my-anchor

**URL when clicked in Excel:**

https://www.domain.tld/some/path/%20-%20my-anchor

This may lead to unexpected behavior on webserver application level e.g. when routing can not be resolved successfully
and the request will results in an 404 error.

The probably best way would be to fix this behavior "somehow" in Excel, but this does not always seem to be possible as
described in [this](https://stackoverflow.com/questions/25070176/hyperlink-changes-from-to-20-20-when-clicked-in-excel)
stackoverflow question.

In order work around this problem for a certain application on a webserver, I added a simple redirect which replaces
the "%20-%20" with a "#" using the following **.htaccess rewrite rule**:

{% highlight apache %}
RewriteRule ^(.*)\ \-\ (.*)$ /$1#$2 [NE,L,R=301]
{% endhighlight %}

This is for sure not a general solution for the problem, put works perfectly when you only have to fix incoming links
for a given application.