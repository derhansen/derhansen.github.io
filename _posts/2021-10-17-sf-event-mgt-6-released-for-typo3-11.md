---
layout: post
title: TYPO3 extension "Event management and registration" version 6.0 for TYPO3 11.5
  LTS released
date: '2021-10-17T15:06:00.001+02:00'
author: Torben Hansen
tags:
- sf_event_mgt
- TYPO3 11.5
modified_time: '2021-10-17T15:20:40.852+02:00'
blogger_id: tag:blogger.com,1999:blog-6517038209122183182.post-7670450650610292820
blogger_orig_url: http://www.derhansen.de/2021/10/sf-event-mgt-6-released-for-typo3-11.html
permalink: /2021/10/sf-event-mgt-6-released-for-typo3-11.html
---

I am really proud and happy to announce, that the new version 6.0. of my TYPO3 extension **"Event management and
registration"** ([GitHub](https://github.com/derhansen/sf_event_mgt)
/ [TYPO3 Extension Repository](https://extensions.typo3.org/extension/sf_event_mgt)) is now fully compatible with 
**TYPO3 11.5 LTS** including support for **PHP 7.4 and 8.0**.

Originally I wanted to release this version of the extension on the same day as TYPO3 11.5 LTS got released, but I
decided to consider all possible deprecations from TYPO3 core and also to reactor the extension to support strict types
and strict properties where ever possible. All in all, my planned 6 days for a TYPO3 11.5 LTS compatible version
resulted in more than 10 days of work. Well, not all changes were required for the release (e.g. removal of
switchableControllerActions), but the code base is now better than before and I'm happy with all improvements that made
its way into the extension.

#### Changes in more than 145 commits

The most important changes are of course those who break existing functionality. Although the new version contains 7
breaking changes and much of the codebase has been changed too, existing users can migrate to the new version with the
least possible manual work.

The list below contains some of the important changes:

* The extension uses strict types and typed properties wherever possible
* switchableControllerActions have been removed. The extension now has 7 individual plugins instead. An **update
  wizard** will migrate existing plugins and settings.
* Data Transfer Objects do not extend AbstractEntity any more
* Native TYPO3 pagination API support for event list view
* Captcha integration has been refactored to support both reCaptcha or hCaptcha
* All possible TYPO3 core deprecations have been handled

All breaking changes have been documented in detail in
the [release notes](https://github.com/derhansen/sf_event_mgt/releases/tag/6.0.0), so existing users know which parts of
the extension need further attention when updating.