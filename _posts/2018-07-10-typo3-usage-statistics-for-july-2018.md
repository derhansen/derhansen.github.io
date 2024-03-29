---
layout: post
title: TYPO3 usage statistics for july 2018
date: '2018-07-10T16:19:00.000+02:00'
author: Torben Hansen
tags:
- statistics
- census
- typo3 version analyzer
- TYPO3
modified_time: '2018-07-10T16:37:41.354+02:00'
blogger_id: tag:blogger.com,1999:blog-6517038209122183182.post-8495660352346919109
blogger_orig_url: http://www.derhansen.de/2018/07/typo3-usage-statistics-for-july-2018.html
permalink: /2018/07/typo3-usage-statistics-for-july-2018.html
---

**TR;DR** - I analyzed 48.146.633 websites for TYPO3 usage - the results with aggregated charts can be
found [here](https://www.t3versions.com/statistics-detail/2).

I'm proud to introduce my latest side project called **t3versions**. It is a Python (Django) web application to
identify, if a website is running TYPO3 and if so, which major version is being used. As a result, TYPO3 websites are
saved to the database in order to create an **aggregated overview** with several charts.

Beside the [live check](https://www.t3versions.com/) in the frontend part, the application also contains a task queue,
which can analyze multiple websites in the background an on multiple servers. I imported a list with 48.146.633 domains
of european websites to the task queue and after 21 days, all websites were analyzed.

As a result, t3versions did find **292.629** websites using TYPO3. Those websites are located on servers with 61.269
individual IP Addresses. The archived results including 6 charts (Version overview, Supported version, TLDs, Webservers,
Countries and SSL) can be found [here](https://www.t3versions.com/statistics-detail/2).

I will reschedule the background check on a regular basis. If someone can provide me a list with e.g. Asian or US
domains, feel free to [contact](mailto:derhansen@gmail.com) me.