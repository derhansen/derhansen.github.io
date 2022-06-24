---
layout: post
title: Laravel 5 and queues using redis - get amount scheduled jobs in queue
date: '2015-05-30T14:29:00.000+02:00'
author: Torben Hansen
tags:
- Laravel 5
- queue
- redis
- amount of jobs
- delete jobs
modified_time: '2015-05-30T14:29:40.302+02:00'
blogger_id: tag:blogger.com,1999:blog-6517038209122183182.post-2854668925827949414
blogger_orig_url: http://www.derhansen.de/2015/05/laravel5-queue-redis-amount-of-jobs.html
permalink: /2015/05/laravel5-queue-redis-amount-of-jobs.html
---

I'm currently digging into Laravel 5 and am using the **queue component** to put time consuming tasks in a queue, so
they can be processed in the background. For performance reasons, I switched from using the **database queue driver** to
use the **redis queue driver**. My application has a dashboard, which shows the amount of scheduled jobs for each queue
in the queue system. Using the database queue driver, I used the following code:

<script src="https://gist.github.com/derhansen/c9d6edd589d337130afd.js"></script>

Using the redis queue driver, Laravel creates a list for each queue on the redis server, so you can easily use the
functions from the [predis](https://packagist.org/packages/predis/predis) package, to interact with the redis server. To
get the **amount of jobs in a queue** with the **redis queue driver**, I use the code as shown below.

<script src="https://gist.github.com/derhansen/90a599e249c28b6258a3.js"></script> 

I also needed the possibility to **remove all jobs in a specific queue** at once in my application. To do so, I used the
code shown below.

<script src="https://gist.github.com/derhansen/5709b8de3194566f5653.js"></script>