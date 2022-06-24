---
layout: post
title: New version of sf_event_mgt released
date: '2015-03-17T08:06:00.000+01:00'
author: Torben Hansen
tags:
- sf_event_mgt
- event management
modified_time: '2015-03-17T08:11:41.667+01:00'
blogger_id: tag:blogger.com,1999:blog-6517038209122183182.post-121040570201614533
blogger_orig_url: http://www.derhansen.de/2015/03/new-version-of-sf-event-mgt-released.html
permalink: /2015/03/new-version-of-sf-event-mgt-released.html
---

The new version 1.0.0 of [sf\_event\_mgt](http://typo3.org/extensions/repository/view/sf_event_mgt) has been released. I
decided not to increase the version number to 0.5.4 but to start with 1.0.0 now, as the extension runs stable in some
projects I maintain and also there is no need to keep the extension in alpha/beta state any more. The new version
contains some small bugfixes in the template files and also comes with some nice new features.

The **double-opt in** feature for the confirmation e-mail can now be configured to be **optional** (
settings.registration.autoConfirmation). If the new setting is activated, registrations by participants will
automatically be confirmed.

For **locations** it is not possible to add **latitude, longitude and a description**. This can be useful, if the
location should be shown on a map.

One feature, that has been requested by multiple users is the **iCalendar** download. For events, it is now possible to
add a link, which downloads the event details as an iCalendar file. To keep things flexible, the iCalendar file is
rendered through **Fluid**, so it is also possible to add own fields to the iCalendar file.

The last new major feature is the possibility to **create a registration for multiple participants** at once. There is a
maximum amount of registrations, which can be configured for each event, so a user is not able to book all available
places for an event at once.

All new features are also described in the [manual](http://docs.typo3.org/typo3cms/extensions/sf_event_mgt/) of the
extension. Additional details for this release can be found [here](https://github.com/derhansen/sf_event_mgt/releases).

Thanks to all users, who gave me feedback end new ideas for the extension!

If you find a bug and want to request a feature, please use
the [issue tracker](https://github.com/derhansen/sf_event_mgt/issues) on GitHub