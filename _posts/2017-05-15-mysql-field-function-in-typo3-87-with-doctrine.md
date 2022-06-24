---
layout: post
title: How to use MySQL FIELD-function in TYPO3 8.7 with Doctrine DBAL
date: '2017-05-15T13:38:00.000+02:00'
author: Torben Hansen
tags:
- MySQL field
- typo3 8.7
- Doctrine DBAL
- Extbase
modified_time: '2017-05-15T13:38:35.968+02:00'
blogger_id: tag:blogger.com,1999:blog-6517038209122183182.post-6072736370634954163
blogger_orig_url: http://www.derhansen.de/2017/05/mysql-field-function-in-typo3-87-with-doctrine.html
permalink: /2017/05/mysql-field-function-in-typo3-87-with-doctrine.html
---

When you want to query an Extbase repository for a list of UIDs or PIDs, you usually add a query constraint like **
query->in('pid', $pidList)** to the query, where $pidList is an array of integers. But what, if you want to control the
sorting of the returned records? Lets assume, you want to select the following list of UIDs **\[5, 3, 4, 1\]** from your
repository and the sorting or the UIDs must **remain**.  
Extbase only has the possibility to sort the query result by a given column either **ascending** or **descending**, so
there is no possibility to control so returned sorting as intended.

In order to resolve the problem, I found this very
helpful [article](https://www.rutschmann.biz/blog/post/extbase-datensaetze-aus-uidlist-sortieren/) from Manfred
Rutschmann, where he describes exactly the same situation and offers a solution for the problem, which works great in
TYPO3 7.6. Sadly the solution does not work in TYPO3 8.7 LTS and fails with an exception, that e.g. column _
tx\_myext\_domain\_model\_mymodel.uid=5_ does not exist.

Since I'm using MySQL as a database engine, I tried to find a solution, where I could use
the [MySQL FIELD-function](https://dba.stackexchange.com/questions/109120/how-does-order-by-field-in-mysql-work-internally)
to apply the special sorting to the ORDER BY clause.

Fortunately Doctrine DBAL was integrated in TYPO3 8.7 LTS, which offers an easy and readable way to construct my query.
I added the following function to my Extbase repository.

<script src="https://gist.github.com/derhansen/7a638cf99f18ca584ac5f67de9e81151.js"></script>

In **line 31** I add my own ORDER BY FIELD clause, where the special sorting is applied

Since the queryBuilder just returns an array of database records (where each records is just an array of returned
fields/values), I use the TYPO3 DataMapper in **line 35** to map the returned rows as objects, so the returned
QueryResult object will contain objects of the type **Mymodel**

The example shown has one little downside, since it only works with a MySQL Database.

If you want to get more details about the Doctrine DBAL integration in TYPO3, make sure to read
the [documentation](https://docs.typo3.org/typo3cms/CoreApiReference/stable/ApiOverview/Database/Index.html).