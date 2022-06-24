---
layout: post
title: TYPO3 Extension "Plain FAQ" released
date: '2019-11-22T15:15:00.000+01:00'
author: Torben Hansen
tags:
- FAQ
- irfaq
- TYPO3
- Symfony Console
modified_time: '2019-11-22T15:15:30.371+01:00'
blogger_id: tag:blogger.com,1999:blog-6517038209122183182.post-849607586137241027
blogger_orig_url: http://www.derhansen.de/2019/11/typo3-extension-plain-faq-released.html
permalink: /2019/11/typo3-extension-plain-faq-released.html
---

Today I released the first [public version](https://extensions.typo3.org/extension/plain_faq) of my latest TYPO3
Extension called "Plain FAQ". The name already says it - the extension is a very simple solution to manage Frequently
Asked Questions in TYPO3. Below follow some of the facts about the Extension:

* Compatible with TYPO3 8.7 and 9.5
* Based on Extbase and Fluid
* Covered with unit and functional tests
* Easy usage for editors
* Uses TYPO3 system categories to structure FAQs by category
* Field for media and files
* Possibility to add related FAQs
* Configurable template layouts for the views
* Automatic cache cleanup when a FAQ article has been updated in backend
* Signal slots to extend the extension with own functionality
* Symfony Console commands to migrate from ext:irfaq

The extension is available on [TER](https://extensions.typo3.org/extension/plain_faq)
and [packagist](https://packagist.org/packages/derhansen/plain-faq).

### Migration from "Modern FAQ (irfaq)"

If you currently use the TYPO3 Extension "Modern FAQ (irfaq)" you may have noticed, that the extension is not compatible
to TYPO3 9.5 (last Extension-Update in March 2018)  and the architecture is quite old (AbstractPlugin, Marker Based
Templates and TypoScript configuration for wraps).

For users where "Modern FAQ (irfaq)" is a blocker for an upcoming TYPO3 9.5 Update, it is possible to migrate to "Plain
FAQ" using the Symfony Console Commands included in "Plain FAQ". The migration is as easy as the usage of the extension:

1\. Migrate existing Categories to sys\_category

2\. Migrate existing FAQs to "plain\_faq" records

3\. Migrate existing Plugins including Plugin settings

The migration may not cover all possible scenarios (e.g. Ratings, Question asked by, irfaq Plugin settings set by
TypoScript), but is for sure a good starting point in order to migrate existing records. I guess, for most websites the
included migration will suite without further work on migrated data.

You can find details about the migration process in the Extension Manual.

### Thanks for sponsoring

I would like to thank [Julius-Maximilians-Universität Würzburg](https://www.uni-wuerzburg.de/) for sponsoring the
initial development of the TYPO3 extension. Thanks for supporting TYPO3 and open source software!