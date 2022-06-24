---
layout: post
title: How to disable the nginx TYPO3 cache set by ext:nginx_cache in development
  context
date: '2019-08-25T19:22:00.000+02:00'
author: Torben Hansen
tags:
- nginx
- disable
- cache
- development
modified_time: '2019-08-25T19:22:56.474+02:00'
thumbnail: https://1.bp.blogspot.com/-WsS2GuNrkyM/XWAktySEtEI/AAAAAAAAhNY/2DsjokFgW94GKihzfvarCMVA9WtIN2LeQCLcBGAs/s72-c/typo3-cache.png
blogger_id: tag:blogger.com,1999:blog-6517038209122183182.post-1356148107491560920
blogger_orig_url: http://www.derhansen.de/2019/08/how-to-disable-typo3-nginx-fastcgi-cache-in-development.html
permalink: /2019/08/how-to-disable-typo3-nginx-fastcgi-cache-in-development.html
---

When you run TYPO3 on nginx webservers, you can use the nginx FastCGI cache to really increase the performance of your
website. Basically, the nginx FastCGI cache stores pages rendered by TYPO3 in the webservers memory. So once a page is
cached in the nginx FastCGI cache, it will be delivered directly from the webservers memory which is really fast.

When using nginx FastCGI cache, the TYPO3 extension [nginx\_cache](https://extensions.typo3.org/extension/nginx_cache/)
can be used to ensure that the nginx FastCGI cache is purged, when TYPO3 content changes. Also the extension ensures,
that pages are removed from the cache, when the TYPO3 page lifetime expires.

However, when you do not need the nginx FastCGI cache while **developing locally** (e.g. no cache configured or even a
different webserver), the clear cache function of TYPO3 results in an error message.

![](/assets/images/2019-08-25/image1.png)

In the TYPO3 log you can find messages like shown below:

{% highlight text %}
Core: Exception handler (WEB): Uncaught TYPO3 Exception: #500: 
Server error: `PURGE https://www.domain.tld/*` resulted in a `500 Internal Server Error`
{% endhighlight %}

The message states, that the PURGE request to the nginx FastCGI cache failed, simply because the PURGE operation is not
allowed or configured.

Since the extension nginx\_cache has no possibility to disable it functionality, you can remove it locally. But if you
have the file PackageStates.php in version control, uninstalling the extension can be error prone, since one by accident
may commit the PackageStates.php without ext:nginx\_cache installed resulting in the cache to be disabled on the
production system.

In order to disable the extensions functionality locally (or in your development environment), you should add the
following to the ext\_localconf.php file of your sitepackage:

{% highlight php %}
// Disable nginx cache for development context
if (\TYPO3\CMS\Core\Utility\GeneralUtility::getApplicationContext()->isDevelopment()) {
    unset($GLOBALS['TYPO3_CONF_VARS']['SYS']['caching']['cacheConfigurations']['nginx_cache']);
    unset($GLOBALS['TYPO3_CONF_VARS']['SC_OPTIONS']['t3lib/cache/frontend/class.t3lib_cache_frontend_variablefrontend.php']['set']['nginx_cache']);
    unset($GLOBALS['TYPO3_CONF_VARS']['SC_OPTIONS']['tslib/class.tslib_fe.php']['pageLoadedFromCache']['nginx_cache']);
}
{% endhighlight %}

This will unset the cache and all hooks set by the extension if TYPO3 runs in development context. Note, that adding the
snippet to **AdditionalConfiguration.php will not work**, since the ext\_localconf.php of ext:nginx\_cache is processed
after AdditionalConfiguration.php.

Finally, you have to ensure, that the new code is processed after the ext\_localconf.php of ext:nginx\_cache is
processed. In order to do so, you have must add **a dependency** to ext:nginx\_cache the ext\_emconf.php of your
sitepackage as shown below:

{% highlight json %}
  'constraints' => [
      'depends' => [
          'nginx_cache' => '2.1.0-9.9.99',
      ],
      'conflicts' => [],
  ]
{% endhighlight %}

I should be noted, that the shown technique can also be used to unset/change a lot of other settings which are made by
installed extensions.