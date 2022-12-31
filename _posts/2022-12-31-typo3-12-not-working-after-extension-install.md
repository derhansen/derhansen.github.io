---
layout: post
title: TYPO3 12 not working after extension install
date: '2022-12-31T10:12:00.005+02:00'
author: Torben Hansen
tags:
- TYPO3
- extension
modified_time: '2022-12-31T10:12:00.005+02:00'
permalink: /2022/12/2022-12-31-typo3-12-not-working-after-extension-install.html
---

While making some of my extensions compatible to TYPO3 v12, I stumbled across the problem, that the whole
TYPO3 website did not work anymore (frontend/backend) after the extension has been installed. Obviously there
was no error message in the logs available, and it took me some time to figure out, why the website was not working.

With TYPO3 11.5, the legacy constant `TYPO3_MODE` was deprecated. My extension did still use the old constant, so
the code `defined('TYPO3_MODE') or die();` resulted in a hard exit of the code in `ext_localconf.php`. After migrating
the code to `defined('TYPO3') or die();`, TYPO3 worked again.
