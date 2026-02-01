---
layout: post
title: 'How to not use the t3versions website'
date: '2026-02-01T12:40.000+02:00'
author: Torben Hansen
tags:
- t3versions
- bot
- headless browser
- api
- ai
modified_time: '2026-02-01T12:40.000+02:00'
permalink: /2026/01/how-not-to-use-t3versions.html
---

## TL;DR: Don't Scrape - Use the API

Instead of building a bot to execute **t3versions** domain checks via a headless browser, just request an API key and 
use it. It’s faster, more reliable, and won’t get you banned.

## What is t3versions?

About eight years ago, I created [t3versions](https://www.t3versions.com/) as a simple tool to identify websites using 
the TYPO3 CMS. When a site is running on TYPO3, the tool attempts to determine and display the major version. In 2020, 
I introduced a free API to allow external applications to perform these checks programmatically. While the API is 
free, it does require an API key, which can be requested via email.

## A sudden spike in traffic

On Thursday, January 29, 2026, I noticed a **high increasement** in check requests. Instead of the typical volume of 100 to 
200 checks per day, the number skyrocketed to approximately 3500.

A quick audit of the access logs revealed the culprit: a bot. Operating across **four different IP addresses** from the 
Hetzner cloud, the bot followed a consistent pattern. It would load the t3versions homepage and then submit a 
POST request for a specific domain. Because the requests were loading all CSS and JS resources, it was clearly 
utilizing a headless browser.

## Hitting rate limits 

The t3versions website uses several security measures, including a **rate limit of 100 requests per day** for standard 
web checks. The bot was evidently oblivious to this limit; it continued to submit checks to the server even after the 
application began returning 403 Forbidden responses.

Furthermore, my server configuration includes secondary rate limits. After repeatedly failing with 403 errors, 
the bot hit these server-level blocks and was locked out of the website entirely.

## Why take the hard road?

I find it confusing that someone would invest the effort to orchestrate a multi-IP bot on a cloud provider just to 
scrape a site via a headless browser. Since a **free API** is readily available, why not use it? Moreover, why program 
a bot to keep retrying once it consistently receives 403 errors?

My best guess is that the user was unaware of the API and perhaps used an AI tool to generate a scraping script. 
AI models often default to browser automation for these tasks if not directed otherwise. While I can’t prove it, 
it seems the most plausible explanation for such an "expensive" yet inefficient approach.