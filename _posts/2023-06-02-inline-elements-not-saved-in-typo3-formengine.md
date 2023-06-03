---
layout: post
title: Invalid custom formengine element causing persistence issue in TYPO3 backend
date: '2023-06-03T09:24:00.000+02:00'
author: Torben Hansen
tags:
- typo3
- formengine
- custom element
modified_time: '2023-06-03T09:24:00.000+02:00'
permalink: /2023/05/invalid-custom-formengine-element-causing-persistence-issue-in-typo3-backend.html
---

**TL;DR:** Always make sure, that a custom TYPO3 formengine element returns _valid HTML_ in `$resultArray['html']`

Some days ago I stumbled across this [issue](https://forge.typo3.org/issues/100943) on TYPO3 forge with the 
title **"TCA IRRE does not save, maybe if there are to many objects"**. Since I also use inline elements in my 
extensions and I never experienced the reported problem, I was in doubt, that the problem was related to 
TYPO3 core, since TYPO3 has not any hard limit for the amount of inline records.

As the issue description was very detailled and the reporter even attached an extension to test the scenario, 
I took some time to analyse the problem.

First I could not really reproduce the problem and assumed, that it maybe was related to PHP `max_input_vars` limits
on the server the website was running. But it came out, that this was not the reason for the problem, since the
reporter increased `max_input_vars` to a very high value and the problem was still present.

I then again tried to reproduce the problem and this time I was successful. I did not even add many inline elements
as suggested in the report, but only added a few elements. So the problem could not be related to the amount of
inline elements at all. I therefore checked the TCA of the affected records and noticed, that a _custom formengine 
element_ was in use in the main record. After I removed the custom formengine element, the problem was gone.

I was however interested in, why the element caused the described problem. After debugging some time, I found out,
that the `POST` parameter `doSave` was not set to `1`, when the custom formengine element was in the TCA. Since the
value of `doSave` is set by JavaScript when the user clicks the save-button in TYPO3 backend, the custom formengine 
element must in some way be invalid. After my analysis, I found the following 2 errors with the element. 

The first problem with the element was, that it used `'type' => 'select'` but without actually holding an input field
for a value. Formengine only has one single type, where no database field is required and that is the `none` field
(https://docs.typo3.org/m/typo3/reference-tca/main/en-us/ColumnsConfig/Type/None/Index.html). 

The second problem was somehow hard to spot, but it was the main reason why new inline elements at some point were
not persisted anymore. The HTML markup in `$resultArray['html']` of the custom formengine element was not valid HTML.
The HTML contained too many closing `div` elements, which resulted in the whole structure of the TYPO3 backend form 
being invalid/unexpected. No error was shown in the browser console, but the JavaScript form validation part of 
formengine was not able to fully validate the form anymore and therefore finally did not set `doSave` to `1` resulting
in changes not being saved.

As a result of the analysis, I created a [pull request](https://github.com/TYPO3-Documentation/TYPO3CMS-Reference-TCA/pull/709/files) 
for the TYPO3 documentation to outline, that custom formengine elements must always return valid HTML to avoid
unexpected behavior in TYPO3 backend. 