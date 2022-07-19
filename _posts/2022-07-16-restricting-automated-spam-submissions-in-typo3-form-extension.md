---
layout: post
title: Restricting automated spam submissions in web forms
date: '2022-07-16T19:20:00.005+02:00'
author: Torben Hansen
tags: 
- form
- spam
- typo3
- ext:form
modified_time: '2022-07-19T12:57:33.005+02:00'
permalink: /2022/07/restricting-automated-spam-submissions-in-web-forms.html
---

TD;DR - Use JavaScript to calculate the value of a hidden field which is evaluated on form
submission. [See example](/2022/07/restricting-automated-spam-submissions-in-web-forms.html#make-automated-form-submissions-harder-using-javascript)

It can be frustrating when spambots automatically submit forms on a website you run. Many years ago, when websites used
to have guestbooks, the spambots usually added new guestbook entries containing links in order to create a huge amount
of backlinks to questionable websites. Today, the intention of most spambots changed. Content filled out in a form (e.g.
contact- or registration form) on a website may get *saved to a database* for further processing or may get *sent to an
email recipient*. For contact forms, the latter is a usual procedure and one of the main advantage for spammers is, that
those emails usually pass spamfilters, since the sending server may be trusted by the recipients email infrastructure.
Knowing this, spammers often just submit "regular" spam content (e.g. short text with link to a website like "Just one
click and your money will grow. https://website.tld") through forms on websites.

### Honeypots, captchas and some other techniques

In order to identify or prevent automated form submissions, some techniques appear to limit the amount of 
spam. 

A honeypot field is a hidden or invisible form field which is used to identify spam submissions. It is efficient,
when a spambot just "blindly" fills out all fields of a form. As soon as the honeypot field is filled out, it is 
clear that the form submission is spam. However, spambots may ignore hidden or invisible fields which basically 
makes the technique ineffective as the "main" spam prevention method.

Captcha fields always require user interaction. The user has to e.g. calculate a simple math task or has to identify
objects on images. This spam prevention method is very effective, but may result in user frustration especially when
the captcha is hard to read/resolve. External 3rd party services like Google reCAPTCHA or hCaptcha offer a stable
and working captcha solutiuon, but may not be inline with local data privacy policies. 

Another possibility to prevent spam in web forms is to analyze submitted data before it is saved or sent to an email
recipient. Such content inspection techniques can be implemented locally (e.g. check submitted data for links, expected
language, IP address) or remotely through a SaaS solution. I would always recommend using a local content inspection, 
since submitting real data to a 3rd party service may not be inline with local data privacy policies.

### Analysis for a contact form created with TYPO3 ext:form

On a TYPO3 website I use ext:form for a simple context form. Usually, the included honeypot field and the hidden 
`__state` field holding the forms state was enough to prevent most automated form submissions. For the initial page
with the contact form (a cached TYPO3 page), the content of the `__state` field usually change depending on the page 
cache lifetime. So for a page lifetime of 24 hours, the initial `__state` value is only valid 24 hours and a spambot
who saved the initial form data will have no luck submitting the form, when the TYPO3 page cache expired and a 
different `__state` data is expected. Some time ago, TYPO3 threw the following bad request exception when the 
`__state` field was invalid:

```
BadRequestException: The HMAC of the form could not be validated.
```

With the latest commits on the [issue](https://forge.typo3.org/issues/90134) in TYPO3 core, this message is not 
logged anymore in TYPO3s `sys_log` or other logs.

However, I recently noticed, that also the `__state` field does not really prevent spambots from automatic form
submissions. Below are some log entries from a successful automated spam submission in ext:form:

```
212.102.40.0 - - [15/Jul/2022:22:20:38 +0200] "GET / HTTP/1.1" 200 4725 "-" "Mozilla/5.0 (Windows NT 6.3; WOW64; rv:59.0) Gecko/20100101 Firefox/59.0"
212.102.40.0 - - [15/Jul/2022:22:20:39 +0200] "GET /kontakt.html HTTP/1.1" 200 4237 "https://domain.tld" "Mozilla/5.0 (Windows NT 6.3; WOW64; rv:59.0) Gecko/20100101 Firefox/59.0
"
212.102.40.0 - - [15/Jul/2022:22:20:40 +0200] "GET /impressum.html HTTP/1.1" 200 3662 "https://domain.tld" "Mozilla/5.0 (Windows NT 6.3; WOW64; rv:59.0) Gecko/20100101 Firefox/59
.0"
212.102.40.0 - - [15/Jul/2022:22:20:44 +0200] "POST /kontakt.html?tx_form_formframework%5Baction%5D=perform&tx_form_formframework%5Bcontroller%5D=FormFrontend&cHash=4057735498895c4baf34
ee7ae746602b HTTP/1.1" 200 5310 "https://domain.tld/kontakt.html" "Mozilla/5.0 (Windows NT 6.3; WOW64; rv:59.0) Gecko/20100101 Firefox/59.0"
```

It seems, the spambot actually visits the main page, waits a second to open the contact form and a different page one 
second later and finally performs a successful form submission 5 seconds after visiting the page with the contact
form. I can only assume how the spambot successfully submitted valid content for the `__state` field, but to me
it seems, that the spambot somehow extracted the value of the hidden `__state` field and submitted it back to TYPO3.

### Make automated form submissions harder using JavaScript

First of all, the following technique is **not a guarantee** to stop automated form spam. Read the end of the articly why. 

Some years ago, I created a so-called "Quick contact form" for a customer, which only contained an input field 
(for phone number) and a textfield (for a message). This form was an easy target for spambots, since the included 
honeypot field could easily be bypassed. I then decided to add a spam protection based on JavaScript, which 
was based on a given challenge on the client and an expected response on the server side. The solutions works 
fine since approximately 4 years now. I therefore created a similar solution for TYPO3 ext:form, which is released
on [packagist](https://packagist.org/packages/derhansen/form_crshield) and 
[TER](https://extensions.typo3.org/extension/form_crshield) and which stopped automated form submissions at least 
for the TYPO3 websites I am responsible for.

The **JavaScript challenge/response spam protection** does not require any user interaction and is pretty simple:

1. A hidden input field with a server side generated data-attribute (the challenge) is added to all forms
2. The client executes the included JavaScript, which:
   * extracts the challenge from the hidden input field
   * calculates the response (ROT13 value of given challenge)
   * adds the calculated response as value to the hidden input field after a given amount of seconds
3. When the form is submitted, the server evaluates the submitted response and invalidates the form submission, 
   if the challenge is not as expected 

In order to make things harder for a spambot, the given challenge has a limited lifetime. For my TYPO3 extension 
ext:form_crshield this lifetime is based on the TYPO3 page cache lifetime. 

For my TYPO3 extension I used ROT13 for the response calculation. You can however use whatever algorithm you want 
for a custom challenge/response form protection, It is also recommended making the challenge as variable as 
possible (e.g. include browser agent string, current date, ...), so the challenge changes from time to time 
and has a limited lifetime.

### Why JavaScript is not THE main solution to prevent form spam  

To my analysis and experiences over the past few years, JavaScript seems to be something that spambots do not master. 
From a technical point of view, this may be a valid assumption, since automating a task requiring DOM analysis and 
JavaScript will consume much more system ressources (e.g. headless browser, selenium, ...) and potentially makes
automation more expensive for spammers.

A spammer targeting explicitly your website can of course easily use reverse engineering to extract the 
challenge/response logic and code something that computes the required calculation in a script. But I assume, that 
for the vast majority of the spammers out there, such an analysis and reverse engineering will be too much effort.
 