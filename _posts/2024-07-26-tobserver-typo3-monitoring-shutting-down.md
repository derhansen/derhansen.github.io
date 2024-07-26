---
layout: post
title: 'tObserver TYPO3 monitoring service shutdown'
date: '2024-07-26T12:14:28.000+02:00'
author: Torben Hansen
tags:
- tObserver
- TYPO3 Monitoring
modified_time: '2024-07-26T14:28:00.000+02:00'
permalink: /2024/07/tobserver-typo3-monitoring-service-shutdown.html
---

**TL;DR:** My free TYPO3 monitoring service [tObserver](https://www.tobserver.com/) will shut down on **1st of August 2024**

### The evolution of tObserver 

Back in 2015 I was working for a company which provided TYPO3 website development and hosting. We had to manage 
hundreds of TYPO3 websites and with each TYPO3 core or extension security update, we had to identify, which of our 
managed TYPO3 websites did use the affected TYPO3 core version or extension, so we could install the security 
updates. This demand led to the idea of tObserver.

On July 9th 2015, I started to develop tObserver in my free time. The initial application was created with one of 
the early versions of [Meteor.js](https://www.meteor.com/), which at that time provided packages for rapid application 
prototyping and development. Some month after the intial development start of tObserver, I received a promising job 
offer, which I finally decided to accept. My new employer had even more TYPO3 websites to manage, but security updates 
and TYPO3 website management did not have the highest priority in that company, so I did not get the chance to test
the service in a bigger scale and mainly used the service for private TYPO3 projects. 

In January 2016 I decided to make the service public available, unlocked the user registration and released the 
initial version of the tObserver TYPO3 extension. The first few users registered after short time and used the 
service. My personal interest in Meteor.js did however decrease and additionally, the framework evolved quickly
and the packages I used, became unsuported and unmaintained. Also, I became a freelancer in 2016, so available
time for the project was limited. The last commit in my Meteor.js project was in October 2016, where I most likely 
also decided to rewrite the whole application from scratch with [Laravel](https://laravel.com/).

The rewrite with Laravel was done by my colleague Stefano Kowalke and me in our free time. We initially planned to 
provide tObserver as a SaaS service, where users had to pay for each managed instance. I however quickly realized,
that forming a company and creating a SaaS service in my free time did not really fit into my daily work as a 
freelancer and my private life, so I decided to slow down and to stop the commercial idea behind tObserver. Instead,
I decided to offer the service for free. 

Since May 2017, the Laravel based tObserver TYPO3 monitoring service was available for free for all users. I personally
used the service to monitor some of my customers TYPO3 websites, which at that time was a helpful solution for me.

### Current state of tObserver in 2024

Since the release of the Laravel based version of tObserver in 2017, many things changed both from a technological and 
from a personal perspective. 

**Personal demand**

While I believe, that there is still a demand for a tool to manage dozends of TYPO3 websites, my personal focus on the
topic in terms of security updates did change. With composer based websites and build pipelines, it is today possible
to let the CI tool of your choice update a TYPO3 website. Scheduled tasks in the CI tool can check for known 
security issues in composer packages and even automatically install the updates. This basically reduced my personal 
need to have a tool, where I can have an overview of TYPO3 websites, which require security updates. Additionally, 
most customer projects I work in and which do not use any CI, do have custom workflows to keep the TYPO3 websites 
secure and up to date. 

**Technical debt**

When tObserver was initially created, composer was not used in the TYPO3 ecosystem. Still today, only every 2nd TYPO3 
website uses composer. But with composer, the handling of version numbers evolved. Initially, I converted TYPO3 core 
and extension version numbers from `major.minor.patch` format to an integer. This integer is used for version 
comparision. With composer however, you can have version strings (e.g. `v1.0.0` or `1.0.0-dev`), which can be used 
in comparisions using additional packages like `composer/semver`. When tObserver however receives a version string
from a composer based website, it tries to convert it to integer and if that fails, saved the version number as zero. 
It would be good to work with version strings in general, but this would require some bigger refactoring tasks.

**Quality Standards**

I work a lot in TYPO3 projects and live up to high code quality standard. I got used to those standards during my
work as a freelancer and contributer to Open Source project, but did not get the time to apply those standards
and required refactorings to tObserver. 

**Outdated techniques**

Besides the topic with the quality standards, the application itself uses outdated techniques like jQuery (I know,
it is not outdated, but I personally do not use it anymore) or Bootstrap 3. 

**Support and user feedback**

Although the service is provided for free and without any support, some users contact me personally asking for new
features, improvements or bugfixes. On one side I can understand the demand behind those requests, but feel sorry, 
when I have to say to users, that I do not have time or motivation to work on the topics. 

**Hosting and Maintenance**

Finally, there is the topic of hosting and maintenance of the application. The applications runs on a small virtual 
machine at Hetzner, for which I pay a small amount of money on a monthly basis. Money is however not really an issue
in this case, but it is the **time** which is required to keep the server and application up to date and secure. The 
Laravel framework has to be updated on a regular basis and the same applies to the servers operating system. This 
is time, which I can not provide, especially in terms of a application, which personally has no use for me anymore.

### Shut down of service 

The past nine years I have spent on my tObserver side project have been fun, interesting, and challenging. I took some
learnings about the lifetime and evolution of a JavaScript Framework, wrong technical decisions and the results of 
personal changes in interests and lack of motivation.

I`ve now been thinking some time about the future of tObserver and came to the conclusion, that missing personal demand, 
missing time and missing motivation are no good factors to continue the service. The service will therefore be shut down 
on 1st of August 2024 and all data will be deleted.

Thanks to the users, who trusted and used the service. I know, that the shutdown of the service may cause you some 
trouble, but be assured, that taking the decision for the shutdown was not easy.

And as a final note: I will not provide the source code of tObserver, even if someone is willing to pay money for it.  
