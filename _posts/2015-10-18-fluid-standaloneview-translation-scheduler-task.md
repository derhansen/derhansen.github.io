---
layout: post
title: TYPO3 - Using Fluid StandaloneView to render localized templates in a scheduler
  task (part 1)
date: '2015-10-18T19:45:00.001+02:00'
author: Torben Hansen
tags:
- fluid
- f:translate
- scheduler task
- TYPO3
modified_time: '2015-11-15T19:48:05.450+01:00'
blogger_id: tag:blogger.com,1999:blog-6517038209122183182.post-804740165641750256
blogger_orig_url: http://www.derhansen.de/2015/10/fluid-standaloneview-translation-scheduler-task.html
permalink: /2015/10/fluid-standaloneview-translation-scheduler-task.html
---

tl;dr: If you want to use Fluid StandaloneView to render a template in a given language from the **_backend context_** (
e.g. scheduler task), make sure you set the language in `$GLOBALS['BE_USER']->uc['lang']`

If you want to **switch the language several times in one request**, please read on
in [part 2](http://www.derhansen.de/2015/11/typo3-using-fluid-standaloneview-to.html) of this blogpost.

**Problem description**

For my Event Management TYPO3 Extension I am developing a feature, where I need to send out **localized e-mails** to
users. The e-mail content is created using a **Fluid StandanloneView** in a TYPO3 **scheduler task**. Actually, this
doesn't work out of the box.

In TYPO3 you can use **Fluid StandaloneView** to render HTML based content, which e.g. can be added to body field of an
HTML e-mail. The TYPO3 Wiki contains
some [code snippets](https://wiki.typo3.org/How_to_use_the_Fluid_Standalone_view_to_render_template_based_emails) on how
this can be processed. If you use the provided code snippets in your ExtBase extension, it works fine for 
**localized content** as long as you use it in the **_frontend context_** of TYPO3 (e.g. website user requests an action).

When you use Fluid StandaloneView from the **_backend context_** to generate localized content, the output differs from
what you would expect. When you are logged in as a TYPO3 backend user, then the localized content is rendered with the
language the **backend user** has chosen in the **user setup**. In addition, if you render a Fluid StandaloneView from
a **commandController**, then the language is **ignored completely**.

**Solution**

After digging into the problem, I found a simple way to control the language which is used during the rendering process
of the Fluid StandaloneView. For the backend context, you just need to set a language (ISO2 code, lowercase) in the 
`$GLOBALS['BE_USER']->uc['lang']` setting. Below follows a code example which shows how this is done (see line 12).

Note, that you also have to make sure, that the Fluid StandaloneView knows, from which extension the localizations get
loaded. This is set in line 24.

The shown method works in both **TYPO3 6.2** and the **current TYPO3 master**. I provided a
little [demo extension](https://github.com/derhansen/standaloneview/releases/tag/0.1.0), which I used for testing
purposes. It contains a backend module and a command controller which renders a Fluid StandaloneView with a given
language.

**Technical background**

The **f:translate viewHelper** uses the TYPO3 LocalizationUtility to render localized content. For the frontend context,
all labels are translated depending on the language set in `$GLOBALS['TSFE']->config['config']['language']`. For
the backend context, the language set in `$GLOBALS['BE_USER']->uc['lang']` is used.