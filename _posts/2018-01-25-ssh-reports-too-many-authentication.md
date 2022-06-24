---
layout: post
title: SSH reports "Too many Authentication Failures" on first connect
date: '2018-01-25T15:44:00.002+01:00'
author: Torben Hansen
tags:
- ssh
- public key
- Too many Authentication Failures
modified_time: '2018-01-25T15:44:32.017+01:00'
blogger_id: tag:blogger.com,1999:blog-6517038209122183182.post-8979592394844216214
blogger_orig_url: http://www.derhansen.de/2018/01/ssh-reports-too-many-authentication.html
permalink: /2018/01/ssh-reports-too-many-authentication.html
---

Today I wanted to connect to a new clients SSH server and received a **"Too many Authentication Failures"** message just
on the first connect to the host. After a short break and some Google research, I found the **very simple reason** for
the message.

Since I have several SSH keys in my **.ssh/** directory, SSH tries to use each of it to connect to the SSH server. So
when the SSH server has a very low "**MaxAuthTries**" setting configured, then the SSH connection may fail before
password authentication is offered.

In order to connect a SSH servers with a low "MaxAuthTries" setting, you can use the following command:

{% highlight shell %}
ssh -o PubkeyAuthentication=no user@host.tld
{% endhighlight %}

After using the "PubkeyAuthentication=no" option, I could login to the host and add a SSH public key to the
.ssh/authorized\_keys file.