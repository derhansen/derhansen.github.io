---
layout: post
title: How to limit the TYPO3 category tree to a subset of categories for extension
  records
date: '2020-04-02T15:26:00.000+02:00'
author: Torben Hansen
tags:
- system categories
- limit
- TYPO3
- subset
modified_time: '2023-05-22T13:22:34.360+02:00'
thumbnail: https://1.bp.blogspot.com/-50bdW9MTdM0/XoTZTIle1hI/AAAAAAAAlC8/TuVy2bBNPZskJWXOGXKTlnECnMNKJGgdwCLcBGAsYHQ/s72-c/typo3-full-category-tree.png
blogger_id: tag:blogger.com,1999:blog-6517038209122183182.post-837956653386698371
blogger_orig_url: http://www.derhansen.de/2020/04/limit-typo3-category-tree-to-subset-for-extensions.html
permalink: /2020/04/limit-typo3-category-tree-to-subset-for-extensions.html
---

In many TYPO3 projects I've been working in, the TYPO3 category system is used to structure content by one or multiple
categories. A typical category tree I often see is build up as shown in the example below:

![Full TYPO3 category tree](/assets/images/2020-04-02/image1.png)

This is a very plain way to create a category tree and the structure in the example is limited to 3 independent main
categories (Events, Products, Staff).

Quite often, the shown example category tree is used systemwide in TYPO3 and all main categories are shown 
for **all record types**. This can be **confusing for editors**, since when you for example want to assign categories 
for e.g. event records, why should one see and be able to select the categories "Products" and "Staff" including all
subcategories?

Fortunately TYPO3 can be configured to limit the category tree for tables to a given root category. As an example, I
limit the shown categories for event records to only "Event" categories. I assume that the category "Events" has the
sys\_category UID 1.

**PageTS example**

{% highlight php %}
TCEFORM.tx_sfeventmgt_domain_model_event.category.config.treeConfig.startingPoints = 1
{% endhighlight %}

In PageTS such configuration options can be set for any record as long as the following configuration path is met: 
`TCEFORM.[tableName].[fieldName].[propertyName]`

The fieldName is usually "categories" or "category", but this can also be different depending on how categories are
implemented in 3rd party extensions.

The PageTS setting can also be set in TCA as shown below.

**TCA example**

{% highlight php %}
$GLOBALS['TCA']['tx_sfeventmgt_domain_model_event']['columns']['category']['config']['treeConfig']['startingPoints'] = 1;
{% endhighlight %}

As a result, the category tree for event records is now limited to the category "Events" and all subcategories.

![TYPO3 category tree limited to a subcategories of one main category](/assets/images/2020-04-02/image2.png)

I think this is way more clear for an editor than it was before. In general, this can be configured for every table in
the TYPO3 TCA (e.g. pages, files, extensions, ...)

~~The configuration only allows to define one UID as root for the category tree. If more flexibility is needed to limiting
the category tree, then `TCEFORM.[tableName].[fieldName].[config][foreign_table_where]` may be the place to add
own custom conditions.~~

Note, that the old configuration option `rootUid` is deprecated since TYPO3 11.5 and that with TYPO3 11.5+
only `startingPoints` should be used. As a bonus, it is possible to select multiple category UIDs as startingpoint.
