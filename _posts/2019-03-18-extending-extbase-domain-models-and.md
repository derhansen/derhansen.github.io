---
layout: post
title: Extending Extbase domain models and controllers using XCLASS
date: '2019-03-18T15:33:00.000+01:00'
author: Torben Hansen
tags:
- "#86270"
- controller
- config.tx_extbase.objects
- model
- extend
- Extbase
modified_time: '2023-03-05T18:34:00.755+01:00'
blogger_id: tag:blogger.com,1999:blog-6517038209122183182.post-6111849216340821268
blogger_orig_url: http://www.derhansen.de/2019/03/extending-extbase-domain-models-and.html
permalink: /2019/03/extending-extbase-domain-models-and.html
---

**TL;DR** - In **TYPO3 12+**, use XCLASS to extend existing extbase domain models and controllers. For **TYPO3 9.5 - 11.5** 
use XCLASS and `registerImplementation()` as shown below.

In TYPO3 9.5 LTS it has been
deprecated ([see notice](https://docs.typo3.org/typo3cms/extensions/core/Changelog/9.5/Deprecation-86270-ExtbaseXclassViaTypoScriptSettings.html))
to extend Extbase classes using TypoScript `config.tx_extbase.objects` and `plugin.tx_%plugin%.objects`. In order
to migrate existing extensions, which extends another TYPO3 extension, you should now use XLASSes.

For my TYPO3 Extension [sf\_event\_mgt](https://github.com/derhansen/sf_event_mgt) I also provide a small demo
extension, which shows how to extend domain models and controllers of the main extension. The previous version using
`config.tx_extbase.objects` can be found [here](https://github.com/derhansen/sf_event_mgt_extend_demo/tree/0.2.0). I
migrated this demo extension to use XCLASSes instead.

The code below shows, how two models and one controller are extended using XCLASS. 

{% highlight php %}
// XCLASS event
$GLOBALS['TYPO3_CONF_VARS']['SYS']['Objects'][\DERHANSEN\SfEventMgt\Domain\Model\Event::class] = [
    'className' => \DERHANSEN\SfEventMgtExtendDemo\Domain\Model\Event::class
];

// Register extended domain class (TYPO3 9.5 - 11.5 only, not required for TYPO3 12)
GeneralUtility::makeInstance(\TYPO3\CMS\Extbase\Object\Container\Container::class)
    ->registerImplementation(
        \DERHANSEN\SfEventMgt\Domain\Model\Event::class,
        \DERHANSEN\SfEventMgtExtendDemo\Domain\Model\Event::class
    );

// XCLASS registration
$GLOBALS['TYPO3_CONF_VARS']['SYS']['Objects'][\DERHANSEN\SfEventMgt\Domain\Model\Registration::class] = [
    'className' => \DERHANSEN\SfEventMgtExtendDemo\Domain\Model\Registration::class
];

// Register extended registration class (TYPO3 9.5 - 11.5 only, not required for TYPO3 12)
GeneralUtility::makeInstance(\TYPO3\CMS\Extbase\Object\Container\Container::class)
    ->registerImplementation(
        \DERHANSEN\SfEventMgt\Domain\Model\Registration::class,
        \DERHANSEN\SfEventMgtExtendDemo\Domain\Model\Registration::class
    );

// XCLASS EventController
$GLOBALS['TYPO3_CONF_VARS']['SYS']['Objects'][\DERHANSEN\SfEventMgt\Controller\EventController::class] = [
    'className' => \DERHANSEN\SfEventMgtExtendDemo\Controller\EventController::class
];
{% endhighlight %}

For domain models in TYPO3 9.5 - 11.5, the important part is the **registerImplementation()** call, since this 
instructs Extbase to use the extended domain model when an object is processed by the property mapper. For TYPO3 12
the **registerImplementation()** call is not required any more.

Note, that there are some limitations using XCLASS, so it is highly recommended to read the
official [documentation](https://docs.typo3.org/typo3cms/CoreApiReference/ApiOverview/Xclasses/Index.html).