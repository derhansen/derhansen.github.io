---
layout: post
title: How to use constructor dependency injection in a XCLASSed TYPO3 class
date: '2021-06-04T16:20:00.003+02:00'
author: Torben Hansen
tags:
- extend controller
- symfony dependency injection
- xclass
- Extbase
- TYPO3
modified_time: '2021-06-04T16:22:16.454+02:00'
blogger_id: tag:blogger.com,1999:blog-6517038209122183182.post-3448183742453201974
permalink: /2021/06/how-to-use-constructor-injection-with-typo3-xclass.html
---

Some time ago in needed to extend an Extbase controller in TYPO3 10.4 which used dependency injection through
constructor injection. So I used XCLASS to extend the original controller and added an own constructor which added an
additional dependency, but this obviously did not work out properly, since the constructor was always called with the
amount of arguments from the original class.

Later I created this [issue](https://forge.typo3.org/issues/91750) on TYPO3 forge in order to find out if this is a
bug/missing feature or if I missed something in my code. In order to demonstrate the problem, I
created [this](https://github.com/derhansen/xclass_di) small demo extension which basically just extended a TYPO3 core
class using XCLASS and just days later, a solution for the issue was provided.

The solution is pretty simple and you just have to ensure to add a reference to the extended class in the Services.yaml
file of the extending extension.

Example:

{% highlight php %}
  TYPO3\CMS\Belog\Controller\BackendLogController: '@Derhansen\XclassDi\Controller\ExtendedBackendLogController'
{% endhighlight %}

The complete `Services.yaml` file can be found [here](https://github.com/derhansen/xclass_di/blob/main/Configuration/Services.yaml).

Thanks a lot to Lukas Niestroj, who [pointed out](https://github.com/derhansen/xclass_di/issues/1) the solution to the problem.