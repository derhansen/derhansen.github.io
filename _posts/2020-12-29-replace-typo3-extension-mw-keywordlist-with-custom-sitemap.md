---
layout: post
title: Replace functionality of TYPO3 extension mw_keywordlist with a custom sitemap
  content element
date: '2020-12-29T11:26:00.000+01:00'
author: Torben Hansen
tags:
- A - Z Keyword list
- content element
- mw_keywordlist
- TYPO3 Extension
- sitemap
modified_time: '2020-12-29T11:26:13.698+01:00'
blogger_id: tag:blogger.com,1999:blog-6517038209122183182.post-5375005890610852709
blogger_orig_url: http://www.derhansen.de/2020/12/replace-typo3-extension-mw-keywordlist-with-custom-sitemap.html
permalink: /2020/12/replace-typo3-extension-mw-keywordlist-with-custom-sitemap.html
---

One of the big pain points when it comes to TYPO3 major updates are extensions. If the extension stack contains
unmaintained / unsupported extensions, updating can really be hard, since existing functionality needs to be replaced
and existing data needs to be migrated.

I recently had this problem on a website, where the TYPO3
extension [mw\_keywordlist](https://extensions.typo3.org/extension/mw_keywordlist/) (A-Z Keyword List) was used. The
project was running on TYPO3 8.7 and was about to be updated to TYPO3 10.4, but an updated version of the extension was
not available, so it was a major blocker in the project.

The extension creates a sitemap generated from keywords in the "pages.keywords" field and renders this sitemap in
alphabetical order grouped by keyword. So basically it creates just another type of sitemap, which TYPO3 already has
other content elements for. In the project team we decided to replace the extension with a 
**custom sitemap content element**, which uses a **custom dataProcessor** to group pages by the configured keywords.

In the sitepackage of the project the following files were added:

* TCA override for tt\_content, so the new content element (CType) "menu\_keywordlist" is registered
* PageTS Config to add the new content element to the content element wizard
* TypoScript to configure rendering of the new sitemap content element
* A custom DataProcessor to group pages by keyword
* A Fluid template to define how the markup is generated

All required files are shown in this [GitLab code snippet](https://gitlab.com/-/snippets/2055043)

After the new sitemap content element has been configured and tested, all existing instances of the mw\_keywordlist
content element were replaced with the new custom sitemap element. This was done using the following SQL query:

{% highlight sql %}
UPDATE `tt_content`
SET `CType` = 'menu_keywordlist'
WHERE `CType` = 'mw_keywordlist_pi1' OR `CType` = 'menu_mw_keywordlist_pi1';
{% endhighlight %}

After the existing content elements were replaced, the extension mw\_keywordlist could be removed. The new solution was
added to the website when it was still running on TYPO3 8.7, since the code is compatible with TYPO3 8.7, 9.5 and 10.4

Thanks to the [University of Würzburg](https://www.uni-wuerzburg.de/) for allowing me to share the code of this
solution.