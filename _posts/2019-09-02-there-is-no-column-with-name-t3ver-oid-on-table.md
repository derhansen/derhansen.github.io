---
layout: post
title: How to fix TYPO3 error "There is no column with name 't3ver_oid' on table"
date: '2019-09-02T16:39:00.000+02:00'
author: Torben Hansen
tags:
- Extbase
- There is no column with name
- dependency
modified_time: '2019-09-02T16:39:19.860+02:00'
blogger_id: tag:blogger.com,1999:blog-6517038209122183182.post-5534423634601352536
blogger_orig_url: http://www.derhansen.de/2019/09/there-is-no-column-with-name-t3ver-oid-on-table.html
permalink: /2019/09/there-is-no-column-with-name-t3ver-oid-on-table.html
---

Recently the following error message showed up in a project I was updating to TYPO3 9.5:

`There is no column with name 't3ver_oid' on table 'tx_news_domain_model_news'.`

When you see this message in a TYPO3 project, you should of course first check, if the field is really available in the
mentioned table and second, you should check, if extension dependencies are correct.

When extending an Extbase extension (in this case ext:news), you must ensure, that the _extending extension_ is **loaded
after** the extension you extend. In order to do so, you must add a dependency to the extension you extend in your
ext\_emconf.php like shown below (example when extending ext:news):

{% highlight json %}
'constraints' => [
    'depends' => [
        'news' => '7.3.0-7.3.99'
    ],
],
{% endhighlight %}

After adding the dependency, make sure to regenerate PackageStates.php ("Dump Autoload Information" in install tool or "
composer dumpautoload"  for composer projects)