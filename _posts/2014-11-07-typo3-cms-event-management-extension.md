---
layout: post
title: TYPO3 CMS - Event management extension with registration option
date: '2014-11-07T15:40:00.000+01:00'
author: Torben Hansen
tags:
- fluid
- TYPO3 CMS
- event registration
- event management
- Extbase
- tdd
modified_time: '2014-11-07T15:40:36.812+01:00'
thumbnail: http://1.bp.blogspot.com/-_1ocWoiiPxE/VFnMUzEm1yI/AAAAAAAANm8/SqobdUK2-1Q/s72-c/event-event.png
blogger_id: tag:blogger.com,1999:blog-6517038209122183182.post-5487338931501658585
blogger_orig_url: http://www.derhansen.de/2014/11/typo3-cms-event-management-extension.html
permalink: /2014/11/typo3-cms-event-management-extension.html
---

Today I've published the first release of my latest extension "**Event management and
registration**" ([sf\_event\_mgt](http://typo3.org/extensions/repository/view/sf_event_mgt)) for TYPO3 CMS. The
extension enables TYPO3 editors to **manage events** in the TYPO3 backend and can show a **list- and a detail-view** for
upcoming, past or all events in the frontend. The list-view can be extended by **individual layouts** as known from the
TYPO3 news extension (tx\_news), so it should for example be possible to create a nice **image slider** for all upcoming
top-events.

As events also could require a **registration process** (e.g. if the number of participants is limited), I added a **
registration option**, so users can register for an event. TYPO3 editors can assign a maximun value for participants to
each individual event, so it can not be **over-booked**. The registration contains a **double opt in** process, so users
must confirm, that they actually have registered for the event. To ensure, that new registrations are processed in time
by the registering user, a **validity for confirmation-links** can be configured.

![Event record in TYPO3 backend](/assets/images/2014-11-07/image1.png)

The extension also comes with a **backend module**, which enables TYPO3 editors to get an overview of all available
events. Also the backend module contains a **CSV-Export** option for events where registration is enabled, so a list of
registered participants easily can be exported.

![Event backend administration module](/assets/images/2014-11-07/image1.png)

The backend module also contains a **notification module**, where TYPO3 editors can send **e-mail notifications** to all
participants of an event (e.g. if the event has been cancelled). **Notification templates** can be configured
individually with some lines of **TypoScript** and with a **Fluid template**.

![Notification module to send notifications to participants](/assets/images/2014-11-07/image1.png)

Finally, the extension has some nice features, which are more technical, but makes the extension very flexible and
extendible. Of course, the extension comes with
a [documentation](http://docs.typo3.org/typo3cms/extensions/sf_event_mgt/Index.html) **ReST format** which explains all
features in detail.

I've tried to keep the **code quality** at a high level, respected the **TYPO3 CGL** as much as possible and also "Event
management and registration" is **fully covered by tests** (which does not mean it is bug-free).

The extension is available on [TER](http://typo3.org/extensions/repository/view/sf_event_mgt) and **bugs** as well as 
**feature requests** can be reported on [GitHub](https://github.com/derhansen/sf_event_mgt), where also the source code of
the extension is hosted.