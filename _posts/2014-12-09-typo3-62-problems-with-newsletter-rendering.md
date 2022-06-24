---
layout: post
title: TYPO3 6.2 - Still some problems with newsletter image rendering
date: '2014-12-09T19:44:00.001+01:00'
author: Torben Hansen
tags:
- Newsletter
- Direct Mail
- Direct Mail Subscription
- TYPO3 6.2
modified_time: '2015-01-01T19:51:59.386+01:00'
thumbnail: http://2.bp.blogspot.com/-04l48_xJwj8/VIbbo9TVqSI/AAAAAAAANv4/ax9Knnii4i4/s72-c/activateContentAdapter.png
blogger_id: tag:blogger.com,1999:blog-6517038209122183182.post-8820177879357927650
blogger_orig_url: http://www.derhansen.de/2014/12/typo3-62-problems-with-newsletter-rendering.html
permalink: /2014/12/typo3-62-problems-with-newsletter-rendering.html
---

I've done a lot of TYPO3 4.5 to TYPO3 6.2 migrations the last weeks and struggled with some strange behaviour of TYPO3
6.2 in combination with **newsletter pages** and the TYPO3 Extensions **direct\_mail** and **
direct\_mail\_subscription**.

I'm pretty sure, some people might run into the same problems during the usage of TYPO3 6.2 and direct\_mail /
direct\_mail\_subscription, so I'll describe how I solved the most common problems in this article.

Update 01.01.2015

The first two problems are [solved](https://github.com/TYPO3/TYPO3.CMS/commit/bce5ae7b988ec2fb658b49a93db946b1ed6e4fd2)
now and you can use renderMethod = table in TYPO3 6.2 and 7.x directly with css\_styled\_content.

### **Newsletter pages not rendering images**

In order to display HTML newsletters in various e-mail clients it is recommended to output the page layout in
old-school **HTML table design**. In order to do so, I use the following TypoScript for images.

{% highlight php %}
tt_content.image.20.renderMethod = table
{% endhighlight %}

This renders image-output in table design and enables the editor to set alignments for images in newsletters. In TYPO3
6.0 and 6.1 I had some major problems with using css\_styled\_content with the renderMethod shown above, so I used to
include the css\_styled\_content configuration for TYPO3 v4.7 as described in my
former [blogpost](http://www.derhansen.de/2013/07/direct-mail-with-images-as-table-output.html).

This all worked fine for TYPO3 sites migrated from 6.x to TYPO3 6.2. With TYPO3 sites, **created directly with TYPO3
6.2**, this did'nt work any more and images were just not rendered. I spent several hours of debugging to find out, that
a setting in the LocalConfiguration.php was the cause for this. Sites, that have been migrated from TYPO3 6.x to TYPO3
6.2 had **\[FE\]\[activateContentAdapter\]** enabled in the Install tool.

After I enabled the \[FE\]\[activateContentAdapter\] for the directly created TYPO3 6.2 sites, images were rendered
correctly with the css\_styled\_content (v4.7) configuration.

![](/assets/images/2014-12-09/image1.png)

Anyway, the install tool shows a **warning**, that setting activateContentAdapter is slow, so this only seems to be a
temporary solution.

In my opinion a better solution would be to **use css\_styled\_content which comes with TYPO3 6.2** and deactivate
\[FE\]\[activateContentAdapter\], but at the time of writing, there is a bug (see problem below) preventing me from
using that option.

### **Image rendering buggy using TYPO3 6.2 css styled content and renderMethod = table**

When I tried to debug my former problem with images not being rendered in page output, I also tried to use
css\_styled\_content from TYPO3 6.2 directly. This resulted in the images being rendered incorrectly (see screenshots
below).

![](/assets/images/2014-12-09/image2.png)

3 TYPO3 Logos in TYPO3 backend - each with an individial label in the image

The output using css\_styled\_content with renderMethod = table results in the following output.

![](/assets/images/2014-12-09/image3.png)

Resulting image output shows 4 images and only the last image is rendered

This seems to be a problem in TYPO3 6.2 and there is also an issue on [Forge](https://forge.typo3.org/issues/47004) for
this one. I will try to create created a patch, so both the linked and related issues can be closed.

### **Newsletter not sent to recipient**

The last problem I spent some time with is, that newly subscribed recipients did not receive the sent newsletter. As I
only send out HTML newsletters, I used to set the following TypoScript in the configuration for **
direct\_mail\_subscription**

{% highlight php %}
plugin.feadmin.dmailsubscription {
  create {
    overrideValues.module_sys_dmail_html = 1
  }
}
{% endhighlight %}

It seems, that this does not work with the latest versions of direct\_mail\_subscription, so newly created records in
tt\_address did not contain the nescecary flag. I tried several other approaches including setting hidden input fields
for the field mod\_sys\_dmail\_html to the registration form and setting TCAdefaults, but all approaches did'nt work. At
least the issue is [known](https://forge.typo3.org/issues/62589), so a fix for this problem may be available soon.

To work around this issue, I set the default value for the field mod\_sys\_dmail\_html to 1 directly in the database by
using the following SQL

{% highlight sql %}
ALTER TABLE `tt_address` CHANGE `module_sys_dmail_html` `module_sys_dmail_html` tinyint(3) unsigned NOT NULL DEFAULT '1';
{% endhighlight %}

Make sure, that this setting may be reverted, if you use the database analyzer in the install tool and apply the
original default value for the field.