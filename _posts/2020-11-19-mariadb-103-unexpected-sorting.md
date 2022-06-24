---
layout: post
title: Unexpected sorting behavior after update from MariaDB 10.1 to 10.3
date: '2020-11-19T20:08:00.002+01:00'
author: Torben Hansen
tags:
- '10.1'
- order by
- mariadb
- '10.3'
modified_time: '2020-11-23T08:09:11.004+01:00'
blogger_id: tag:blogger.com,1999:blog-6517038209122183182.post-1966801751255806775
blogger_orig_url: http://www.derhansen.de/2020/11/mariadb-103-unexpected-sorting.html
permalink: /2020/11/mariadb-103-unexpected-sorting.html
---

**TL;DR** The sorting behavior changed from MariaDB 10.1 to 10.2 due to
a [bug](https://jira.mariadb.org/browse/MDEV-11320) in MariaDB 10.1

After updating from Ubuntu 18.04 LTS to 20.4 LTS a previously working a PHP application which contains a data export
suddenly did not return the expected result any more. I debugged this scenario by comparing the database query results
in the data export and obviously, something in the sorting changed from MariaDB 10.1 to MariaDB 10.3

In order to fully reproduce the problem, I created a really simple use case as shown in the SQL dump below.

{% highlight sql %}
CREATE TABLE `test` (
  `a` int(11) NOT NULL AUTO_INCREMENT,
  `b` varchar(255) NOT NULL,
  `c` text NOT NULL,
  `d` varchar(255) NOT NULL,
  PRIMARY KEY (`a`)
) ENGINE=InnoDB AUTO_INCREMENT=6 DEFAULT CHARSET=utf8;

INSERT INTO `test` VALUES (1,'A','\r\n','CRLF'),(2,'A','',''),(3,'A','',''),(4,'A','\r\n','CRLF'),(5,'A','','');
{% endhighlight %}

So we have a really simple table structure with some data. The only special thing is, that 2 values in column "c"
contain a carriage return and line feed (CRLF). Since this is not printed when selecting data, I also added column d
which contains the value "CRLF" for rows, where column c is a CRLF.

So now I select some data.

**SELECT \* FROM test;**

{% highlight markdown %}
| a | b | c  | d    |
+---+---+----+------+
| 1 | A |    | CRLF |
| 2 | A |    |      |
| 3 | A |    |      |
| 4 | A |    | CRLF |
| 5 | A |    |      |
+---+---+----+------+
{% endhighlight %}

This result is as I would expect it. Now sorting comes into the game...

### Ubuntu 18.04 with MariaDB 10.1.47

**SELECT \* FROM test ORDER BY b ASC, c ASC, a ASC;**

{% highlight markdown %}
+---+---+----+------+
| a | b | c  | d    |
+---+---+----+------+
| 2 | A |    |      |
| 3 | A |    |      |
| 5 | A |    |      |
| 1 | A |    | CRLF |
| 4 | A |    | CRLF |
+---+---+----+------+
{% endhighlight %}

OK, so the sorting of column c puts the CRLF values at the end for MariaDB 10.1. Now I try the same on another system.

### Ubuntu 20.04 with MariaDB 10.3.25

**SELECT \* FROM test ORDER BY b ASC, c ASC, a ASC;**

{% highlight markdown %}
+---+---+----+------+
| a | b | c  | d    |
+---+---+----+------+
| 1 | A |    | CRLF |
| 4 | A |    | CRLF |
| 2 | A |    |      |
| 3 | A |    |      |
| 5 | A |    |      |
+---+---+----+------+
{% endhighlight %}

As you notice, the sorting for column c is now reversed...

I did not find a setting in MariaDB 10.3 to switch back to the sorting as it was in MariaDB 10.1. I could also reproduce
the same behavior on MySQL 8.0. So... bug or feature - who knows? I think the described scenario can be considered as an
edge case, but if you somehow depend on, that sorting for a column with CRLF values is exactly the same, this can hit
you really hard.

I created an [issue](https://jira.mariadb.org/browse/MDEV-24250) in the MariaDB bug tracker. I'm curious if this is
supposed behavior or not.

**Update 23.11.2020:** It has been confirmed, that the sorting behavior is as expected in MariaDB 10.2+ and that it was
wrong in 10.1