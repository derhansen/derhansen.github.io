---
layout: post
title: TYPO3 extension sf_event_mgt version 3.0 released
date: '2018-04-11T08:11:00.001+02:00'
author: Torben Hansen
tags:
- fluid
- event management
- Extbase
- TYPO3
modified_time: '2018-04-11T08:11:58.441+02:00'
thumbnail: https://1.bp.blogspot.com/-qRIxBxIYWtU/WskAtRDSy2I/AAAAAAAAZTk/ssMPW3MKFCEb4hDxASSsFZhnFmtInP6VACLcBGAs/s72-c/event-registrationfields.png
blogger_id: tag:blogger.com,1999:blog-6517038209122183182.post-3488171256725666414
blogger_orig_url: http://www.derhansen.de/2018/04/sf-event-mgt-version-3-released.html
permalink: /2018/04/sf-event-mgt-version-3-released.html
---

Today I finally released the new version 3.0 of my TYPO3 extension **sf\_event\_mgt - Event management and
registration**. The new version comes with tons of new features, bugfixes and improvements and also contains 2 breaking
changes, so make sure to read the [release notes](https://github.com/derhansen/sf_event_mgt/releases/tag/3.0.0).

Thanks to everyone, who contributed to the extension over the last few months. Also a special thanks to **Alex Kellner**
for his extension [powermail](https://extensions.typo3.org/extension/powermail/), from which I adapted ideas and some
code for the **registration fields** feature.

**New features**

Below follows some of the new features of sf\_event\_mgt 3.0.  
**Registration fields**

Im order to make the extension more easy to use for non-programmers, I added the possibility for editors add additional
registration fields to the default registration form on event basis.

![](/assets/images/2018-04-11/image1.png)

**Registration fields**

Creating of registration fields works basically as in powermail. The user can add registrations fields in a new tab as
shown in the screenshot above and choose one of the following field types: **input, textarea, radio and checkbox**. When
a participant registers to an event, all filled out registration fields are saved to the registration record in the
TYPO3 backend.

![](/assets/images/2018-04-11/image2.png)

**allowLanguageSynchronization for TYPO3 8.7**

The TCA settings allowLanguageSynchronization has been introduced in TYPO3 8.6 and is the successor for "
l10n\_mode=mergeIfNotBlank". Note, that sf\_event\_mgt did not use "l10n\_mode=mergeIfNotBlank", so the language
synchronization feature is only available in TYPO3 8.7

![](/assets/images/2018-04-11/image3.png)

**Signals**

Finally, I also
added [signals](https://docs.typo3.org/typo3cms/extensions/sf_event_mgt/ForDevelopers/Signals/Index.html) in various
actions, so it is now easily possible to modify/extend variables given to the desired views.