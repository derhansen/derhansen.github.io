---
layout: post
title: TYPO3 - Adding direct mail fields to femanager
date: '2017-01-25T11:33:00.001+01:00'
author: Torben Hansen
tags:
- Direct Mail
- TYPO3
- femanager
modified_time: '2017-01-25T12:11:04.122+01:00'
thumbnail: https://3.bp.blogspot.com/-Nm2pxPca85A/WIemyDvBcxI/AAAAAAAATaU/5-FoD6itU2EVIR_PLXa68LTLrDSQ3HQlwCLcB/s72-c/femanager-plugin-settings.png
blogger_id: tag:blogger.com,1999:blog-6517038209122183182.post-6964604382045735056
blogger_orig_url: http://www.derhansen.de/2017/01/typo3-adding-direct-mail-fields-to.html
permalink: /2017/01/typo3-adding-direct-mail-fields-to.html
---

In a project I needed to add the **direct mail fields** "Activate Newsletter" and "Subscribe to categories" to the TYPO3
femanager (thanks to Alex Kellner for this great extension), so frontend users are able to subscribe to a newsletter and
select newsletter categories.

Since femanager is created with Extbase, it is easily extendable and I created a small TYPO3 extension
named [femanager\_dmail\_subscribe](https://typo3.org/extensions/repository/view/femanager_dmail_subscribe), that
automatically adds "Newsletter subscription", "Newsletter category" and "HTML newsletter" direct mail fields to
femanager, so editors can select the direct mail fields a frontend user should be able to edit.

![femanager plugin field settings](/assets/images/2017-01-25/image1.png)

In order to display direct mail categories in the frontend, it is required to add the sysfolder with the direct mail
categories to the femanager record storage page.

![femanager plugin record storage page](/assets/images/2017-01-25/image2.png)

After installing and configuring the extension as shown above, frontend users can edit the selected fields (see
screenshot below)

![Frontend user can edit direct mail fields](/assets/images/2017-01-25/image3.png)

I uploaded 2 versions of the extension to [TER](https://typo3.org/extensions/repository/view/femanager_dmail_subscribe).
Version 1.0.0 is compatible with TYPO3 6.2 and femanager 1.5.2 and version 2.0.0 is compatible with TYPO3 7.6 and
femanager 2.x