---
layout: post
title: How to use multiple SMTP accounts in one TYPO3 installation
date: '2022-01-21T13:18:00.003+01:00'
author: Torben Hansen
tags:
- multiple
- TYPO3
- smtp
modified_time: '2022-01-21T15:32:49.102+01:00'
blogger_id: tag:blogger.com,1999:blog-6517038209122183182.post-4351872335620232499
blogger_orig_url: http://www.derhansen.de/2022/01/multiple-smtp-accounts-in-typo3.html
permalink: /2022/01/multiple-smtp-accounts-in-typo3.html
---

When TYPO3 is used to serve multiple websites in one installation, it may sometimes be required to configure multiple SMTP accounts in order to send emails from TYPO3 (e.g. mailforms or notifications) to different recipients. This may especially be important, when the recipient mailserver has a **strict spam filter** or when the domain uses a **SPF, DKIM or DMARC** and the mailserver only accepts emails from thrusted sources.

In TYPO3 you can configure one global SMTP server in **LocalConfiguration.php** by using the following settings:

{% highlight php %}
$GLOBALS['TYPO3_CONF_VARS']['MAIL']['transport'] = 'smtp';
$GLOBALS['TYPO3_CONF_VARS']['MAIL']['transport_smtp_server'] = 'your.mailserver.tld';
$GLOBALS['TYPO3_CONF_VARS']['MAIL']['transport_smtp_encrypt'] = true;
$GLOBALS['TYPO3_CONF_VARS']['MAIL']['transport_smtp_username'] = 'username';
$GLOBALS['TYPO3_CONF_VARS']['MAIL']['transport_smtp_password'] = 'password';
$GLOBALS['TYPO3_CONF_VARS']['MAIL']['defaultMailFromAddress'] = 'email@your.mailserver.tld';
{% endhighlight %}

This setting however reflects to any hosted website in your TYPO3 installation and the email-server for _typo3-website1.tld_ may possible not accept emails with a sender from the domain _typo3-website2.tld_.

In order to provide multiple SMTP servers for different websites in a TYPO3 installation, I configure different SMTP servers in **AdditionalConfiguration.php** 

{% highlight php %}
if (($_SERVER['SERVER_NAME'] ?? '') === 'typo3-website1.tld') {
    $GLOBALS['TYPO3_CONF_VARS']['MAIL']['transport_smtp_server'] = 'mail.typo3-website1.tld';
    $GLOBALS['TYPO3_CONF_VARS']['MAIL']['transport_smtp_encrypt'] = true;
    $GLOBALS['TYPO3_CONF_VARS']['MAIL']['transport_smtp_username'] = 'username';
    $GLOBALS['TYPO3_CONF_VARS']['MAIL']['transport_smtp_password'] = 'password';
    $GLOBALS['TYPO3_CONF_VARS']['MAIL']['defaultMailFromAddress'] = 'email@typo3-website1.tld';
}

if (($_SERVER['SERVER_NAME'] ?? '') === 'typo3-website2.tld') {
    $GLOBALS['TYPO3_CONF_VARS']['MAIL']['transport_smtp_server'] = 'mail.typo3-website2.tld';
    $GLOBALS['TYPO3_CONF_VARS']['MAIL']['transport_smtp_encrypt'] = true;
    $GLOBALS['TYPO3_CONF_VARS']['MAIL']['transport_smtp_username'] = 'username';
    $GLOBALS['TYPO3_CONF_VARS']['MAIL']['transport_smtp_password'] = 'password';
    $GLOBALS['TYPO3_CONF_VARS']['MAIL']['defaultMailFromAddress'] = 'email@typo3-website2.tld';
}
{% endhighlight %}

Since AdditionalConfiguration.php is evaluated on every request, TYPO3 will conditionally use the email settings
depending on the `$_SERVER['SERVER_NAME']` variable.

**Note**, that this solution only applies to web requests and **does not work in CLI context** (e.g. scheduler task).
