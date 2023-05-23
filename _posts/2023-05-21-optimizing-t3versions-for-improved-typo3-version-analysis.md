---
layout: post
title: Optimizing t3versions for improved TYPO3 version analysis
date: '2023-05-21T19:00:00.000+02:00'
author: Torben Hansen
tags:
- t3version
- crawler
- optimization
modified_time: '2023-05-21T19:00:00.000+02:00'
permalink: /2023/05/optimizing-t3versions-for-improved-typo3-version-analysis.html
---

My [t3versions](https://www.t3versions.com) TYPO3 version analysis and statistics service is running for over five years now. 
During the years, I had to learn, that crawling and analyzing most likely the whole WWW for TYPO3 websites is sometimes 
challenging. 

In order to find new websites using TYPO3, I regulary perform a **crawling** process, which checks over **260 million 
domains** for possible TYPO3 usage. With my current infrastructure (3 servers with 6 CPUs each), this task takes about 
14 days to analyze all domains. The crawling process checks the content of a website for TYPO3 fingerprints. When a 
TYPO3 website has been identified, the domain is queued for a detailed analysis using the t3versions API. The 
crawling process runs multicore and multithreaded and consumes **~4TB of traffic**. It is not uncommon, that some 
webservers may _block the t3versions crawling requests_, if a higher amount of GET requests are performed from the 
same IP address in a short amount of time.

The detailed TYPO3 analysis using the [t3versions API](https://www.t3versions.com/api/docs) will result in several GET 
requests to a given website and uses **fingerprinting** techniques, if the TYPO3 major version could not be determined 
by performing the most common checks. Fingerprinting will however also lead to an _unusual amount of GET requests_ 
possibly resulting in a 404 response, which might trigger **WAF** (Web Application Firewall) systems to block the 
requesting IP address for a given time.

Since both the t3versions crawling process and the TYPO3 detailed analysis are performed by the _same servers_, chances
are high, that the IP addresses of my analysis infrastructure might get blocked. I noticed this happended quite often
during the recent t3versions crawling and rescan tasks, so I had to find solution. 

In order to avoid being blocked by WAFs (too quickly), I optimized the TYPO3 analysis process as following:

* The crawling process will use different IP addresses than the detailed TYPO3 analysis use
* TYPO3 websites will not be removed from the database, if a WAF system is detected. Instead, they will be rescanned using a different IP address
* Do not use fingerprinting analysis for websites which are known to use a WAF
* Reduce the amount of GET requests which might result in a 404 error by limiting fingerprinting to TYPO3 major versions only (no sprint release analysis anymore)
* Added a small wait time (random value < 1 second) between fingerprinting requests
* Centralize the GET request logic for all analysis tasks, so it is possible to use proxy servers
* Ensure that GET requests for a website analysis does not always come from the same IP address by using random proxy servers
* Use a pool of private squid proxy servers when rescanning known TYPO3 websites. Costs are quite low (~1€ for the squid servers per rescan)

I am very satisfied with the results of my optimizations, as they have led to a significant decrease in the number 
of false positives (website being removed from the database allthough it uses TYPO3). Through constant efforts and 
fine-tuning, I have achieved a substantial improvement in the accuracy of the system.