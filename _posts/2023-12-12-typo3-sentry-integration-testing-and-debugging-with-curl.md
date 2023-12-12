---
layout: post
title: 'TYPO3 Sentry Integration: Testing and Debugging with cURL'
date: '2023-12-12T18:57:00.000+02:00'
author: Torben Hansen
tags:
- sentry
- typo3
- debugging
modified_time: '2023-12-12T18:57:00.000+02:00'
permalink: /2023/10/typo3-sentry-integration-testing-and-debugging-with-curl.html
---

Configuring the TYPO3 extension sentry_client is a straightforward process that ensures seamless integration 
with Sentry, the robust error tracking and monitoring platform. Once the sentry_client extension is configured, 
it becomes crucial to verify whether Sentry is successfully receiving events from your TYPO3 instance. The 
documentation describes exactly, how to test this by using the following TypoScript snippet:

```
page = PAGE
page.20 = USER
page.20 {
  userFunc = Networkteam\SentryClient\Client->captureException
}
```

This will result in TYPO3 throwing an exception, since the called `captureException` function expects
arguments, which are not provided. So far, so good. But what, if Sentry does not recieve the expected event?

### Creating a test event in Sentry using curl

To ensure the server's ability to connect to Sentry and generate test events, the versatile cURL command can 
be employed as a valuable tool. cURL facilitates the simulation of HTTP requests, making it an effective means 
to verify the communication between your server and the Sentry platform.

```shell
curl -X POST --data '{ "exception": [{ "type": "Sentry curl test request ", "value": "This is a test request to sentry sent by curl"}] }' \
	-H 'Content-Type: application/json' \
	-H "X-Sentry-Auth: Sentry sentry_version=7, sentry_key=<sentry-key>, sentry_client=raven-bash/0.1" https://<sentry-url>/api/<project-id>/store/
```

Replace the following values:

* `<sentry-key>` with the client key. This is usually the part after the DSN schema
* `<sentry-url>` with the URL to your sentry instance
* `<project-id>` with the ID of your project

If the creation of the test event was successful, you will receive an event id from sentry as response as shown below:

```shell
curl -X POST --data '{ "exception": [{ "type": "Sentry curl test request ", "value": "This is a test request to sentry sent by curl"}] }' \                                                                                          127 ↵
        -H 'Content-Type: application/json' \
        -H "X-Sentry-Auth: Sentry sentry_version=7, sentry_key=123456789012345678901234567890, sentry_client=raven-bash/0.1" https://my.sentry.tld/api/1/store/
{"id":"08f0071b965347eca2c6ad4f70c37328"}%
```

A look in the Sentry UI for the project will then also prove, that the event has been created successfully 

![Sentry test event](/assets/images/2023-12-12/sentry-curl-test.png)

### What if the Sentry UI does not show the expected event?

If the Sentry UI does not show the event for a new created project, most likely something with the logging in 
Sentry is wrong. If you use Sentry on premise, try to restart sentry, so all docker containers are restarted. 
