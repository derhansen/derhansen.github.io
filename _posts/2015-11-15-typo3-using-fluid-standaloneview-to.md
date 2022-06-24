---
layout: post
title: TYPO3 - Using Fluid StandaloneView to render localized templates in a scheduler
  task (part 2)
date: '2015-11-15T19:43:00.001+01:00'
author: Torben Hansen
tags:
- fluid
- f:translate
- scheduler task
- TYPO3
modified_time: '2015-11-15T19:44:43.869+01:00'
thumbnail: http://3.bp.blogspot.com/-4RDvRxaXRuY/VkjKiW2kEZI/AAAAAAAAQBg/FAaBCp_-G_0/s72-c/Bildschirmfoto%2B2015-11-15%2Bum%2B19.09.55.png
blogger_id: tag:blogger.com,1999:blog-6517038209122183182.post-2339317341440079580
blogger_orig_url: http://www.derhansen.de/2015/11/typo3-using-fluid-standaloneview-to.html
permalink: /2015/11/typo3-using-fluid-standaloneview-to.html
---

Today I found, that the technique I described in my blogpost
about [rendering localized Fluid templates in a scheduler task](http://www.derhansen.de/2015/10/fluid-standaloneview-translation-scheduler-task.html)
does not work as expected. As long as you want to switch the language used to render the templates only **one time**,
then you're fine. But as soon as you want to switch the language **several times** (e.g. sending multiple localized
e-mails in one request), then you experience that only the first language switch is respected.

The root cause for this is the TYPO3 LocalizationUtility, which includes the static **translate()** method that is used
return translated language labels from XLF/XML language files. The LocalizationUtility is not designed to handle
multiple language switches in **one request**, so at this point I'm stuck.

I order to keep things simple for the integrator (use one e-mail template with language labels to send out localized
e-mails in a scheduler task), I decided to create an own viewHelper which uses a _modified version_ of the
LocalizationUtility. The modified version of the LocalizationUtility does not contain any static variables or methods
and can be used with dependency injection. You can find the code in
this [GitHub repository](https://github.com/derhansen/standaloneview).

**Usage**

In my Fluid StandaloneView templates I now use my own translate viewHelper as shown below.

<script src="https://gist.github.com/derhansen/94e4d055b3bec0217a9b.js"></script>

The viewHelper uses
the [LocalizationService](https://github.com/derhansen/standaloneview/blob/master/Classes/Service/LocalizationService.php) (
which is the TYPO3 LocalizationUtility with some small modifications - e.g. removed all "static" declarations). As a
result of this, all functionality of the original viewHelper / TranslationUtility are remained (e.g. overwriting
language labels with TypoScript)

<script src="https://gist.github.com/derhansen/52d62a63f6802aea4697.js"></script>

I extended the original demo extension from my first blogpost so it makes use of the new viewHelper /
LocalizationService. The Extension now includes a form, which renders multiple Fluid StandaloneViews in one request and
the language is switched for each individual StandaloneView (see result-section in the screenshot below)

![](/assets/images/2015-11-15/image1.png)

The demo extension also includes a command controller, which includes a command that also renders multiple
standaloneViews in one request (see screenshot below)

![](/assets/images/2015-11-15/image2.png)

If I don't find any major problems, I will make use of this technique to send out multilingual e-mails in a scheduler
task in my [Event Management Extension](http://typo3.org/extensions/repository/view/sf_event_mgt).  

**Final notice**

The technique shown should only be used in the **backend context** of TYPO3 when you want to render multilingual Fluid
StandaloneViews **in one request**. I'm not very happy with the approach of "just" taking some code from the TYPO3 core
and adapting it to my needs, since this is not always a clean solution and it may include some drawbacks.