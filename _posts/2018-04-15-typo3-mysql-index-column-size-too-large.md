---
layout: post
title: TYPO3 MySQL database import fails with "Index column size too large. The maximum
  column size is 767 bytes"
date: '2018-04-15T15:08:00.000+02:00'
author: Torben Hansen
tags:
- mysql 5.7
- Index column size too large
- TYPO3
- mysql 5.6
modified_time: '2018-04-15T15:08:45.610+02:00'
blogger_id: tag:blogger.com,1999:blog-6517038209122183182.post-2040738043702597403
blogger_orig_url: http://www.derhansen.de/2018/04/typo3-mysql-index-column-size-too-large.html
permalink: /2018/04/typo3-mysql-index-column-size-too-large.html
---

I recently migrated a TYPO3 7.6 Website to TYPO3 8.7 and while importing the migrated TYPO3 database on the production
server, the import failed with the following MySQL error:

_ERROR 1709 (HY000) at line 2060: Index column size too large. The maximum column size is 767 bytes._

The error occurred for the import of the TYPO3 table sys\_refindex. After some research and local debugging I found out,
that I locally was using MySQL 5.7 and the production server was using MySQL 5.6, but settings in regard to
innodb\_large\_prefix and innodb\_file\_format were equal.

In order to import the dump from my MySQL 5.7 server to the production MySQL 5.6 Server, I executed the following SQL
query before creating the MySQL dump:

{% highlight sql %}
ALTER TABLE sys_refindex ROW_FORMAT=DYNAMIC;
{% endhighlight %}

After setting the ROW\_FORMAT to DYNAMIC, the database dump from MySQL 5.7 could finally be imported without errors on
the MySQL 5.6 production server.