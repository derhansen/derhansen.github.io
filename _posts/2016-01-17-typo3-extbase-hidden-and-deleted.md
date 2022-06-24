---
layout: post
title: TYPO3 ExtBase - Hidden and deleted property of domain models
date: '2016-01-17T20:29:00.001+01:00'
author: Torben Hansen
tags:
- Deleted
- Extbase
- Hidden
- TYPO3
modified_time: '2016-01-27T19:42:08.103+01:00'
blogger_id: tag:blogger.com,1999:blog-6517038209122183182.post-4457055809154316057
blogger_orig_url: http://www.derhansen.de/2016/01/typo3-extbase-hidden-and-deleted.html
permalink: /2016/01/typo3-extbase-hidden-and-deleted.html
---

TYPO3 ExtBase repositories do by default only return **non-hidden** and **non-deleted** records. You can easily override
this behavior by modifying the default query settings for the given repository. If you for example want to return all
FrontendUser (including hidden and deleted) from a TYPO3 website, you can set the query settings as shown below.

Now all available FrontendUser records are returned. When you now loop over the queryResult, you will see, that the
FrontendUser Objects returned **do not contain any information about if the record actually is hidden or deleted**.

<script src="https://gist.github.com/derhansen/c24392b434bacded2b16.js"></script>

_Note that the example above shows the assignment of the defaultQuerySettings directly in an action. Typically you
define this in the initializeObject method or directly in the repository._  
In order to obtain those information, you need to **extend the existing FrontendUser** domain model with two fields.
First, you need to create an own domain model with two properties as shown below (must be located in a extension -
usually your own).  

<script src="https://gist.github.com/derhansen/e061663bfff063e0dda1.js"></script>

Note that I called the property which indicates if a record is hidden or not "disable" and not "hidden". I could not get
the example to work with a property called "hidden".

Next you need to add the new fields to the TCA by adding the file fe\_users.php with the following content to the
folder **Configuration/TCA/Overrides**.

<script src="https://gist.github.com/derhansen/5a68e54c7dc056f981a8.js"></script>

Finally you must add you new domain model as a **subclass** for the original FrontendUser domain modal and **map** the
two new properties. Add the file ext\_typoscript\_setup.txt to the root folder of your extension with the following
content.

<script src="https://gist.github.com/derhansen/589c29acac8a3eeab7db.js"></script>

Note, that the key in line 6 (in this example "0") must be used in line 12 as recordType.

After clearing all caches, ExtBase now returns all FrontendUsers containing the two new properties "**deleted**" and "**
disable**"