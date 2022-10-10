---
layout: post
title: Problem with all extbase plugins during TYPO3 11.5 update
date: '2022-10-09T14:34:00.005+02:00'
author: Torben Hansen
tags:
- TYPO3
- plugin
- extbase
- configurePlugin
modified_time: '2022-10-10T07:20:00.005+02:00'
permalink: /2022/10/problem-with-all-extbase-plugins-during-typo3-11-update.html
---

In a TYPO3 project, where I was updating the website from TYPO3 10.4 to version 11.5, I suddenly faced the situation,
that **all extbase plugins** did not work anymore. The following error message was displayed (example for TYPO3 ext:news):

```
The default controller for extension "News" and plugin "Pi1" can not be determined.
Please check for TYPO3\\CMS\\Extbase\\Utility\\ExtensionUtility::configurePlugin() in your ext_localconf.php.
```

Since all extbase extensions were active and did work in TYPO3 10.4, the problem was most likely related to the
TYPO3 11.5 update. After some time of debugging, I finally found the cause of the problem. One custom extension
in the project had a **PSR-15 middleware**, which provided an API to retrieve data from extbase records. The middleware
was registered before the TYPO3 page resolver middleware `typo3/cms-frontend/page-resolver` and used constructor
injection to inject an extbase repository like shown in the simplified example below:

{% highlight php %}
protected FooRepository $fooRepository;

public function __construct(FooRepository $fooRepository)
{
    $this->fooRepository = $fooRepository;
}

public function process(ServerRequestInterface $request, RequestHandlerInterface $handler): ResponseInterface
{
    // Do API work
}
{% endhighlight %}

Obviously, the constructor injection was responsible for the strage behaviour with all extbase plugins, so I
removed it and used `GeneralUtility::makeInstance()` to create the `fooRepository` object as shown below:

{% highlight php %}
protected FooRepository $fooRepository;

public function process(ServerRequestInterface $request, RequestHandlerInterface $handler): ResponseInterface
{
    $this->fooRepository = GeneralUtility::makeInstance(FooRepository::class);
    // Do API work
}
{% endhighlight %}

After I changed the code as shown, the problem was gone. I did not have time to dig deeper into the problem,
but I guess it may have something to do with the extbase `ConfigurationManager` being injected too early in
the TYPO3 bootstrap. 