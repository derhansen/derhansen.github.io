---
layout: post
title: TYPO3 7.3 - Extending the backend login form by using the new backend login
  form API
date: '2015-06-16T14:16:00.000+02:00'
author: Torben Hansen
tags:
- YubiKey
- TYPO3 7.3
- loginProvider
- backend login form API
modified_time: '2015-06-16T14:17:39.019+02:00'
thumbnail: http://4.bp.blogspot.com/-h1Yi7osOCNI/VXLWaxDSflI/AAAAAAAAOvc/pFTD7OQl8W8/s72-c/openid-link.png
blogger_id: tag:blogger.com,1999:blog-6517038209122183182.post-5787933766727164914
blogger_orig_url: http://www.derhansen.de/2015/06/extending-backend-login-form-in-typo3-7.3.html
permalink: /2015/06/extending-backend-login-form-in-typo3-7.3.html
---

Back in january 2015 I added support for **TYPO3 7.1** to the **YubiKey TYPO3 extension** and raised the extension
compatibility level to be compatible with TYPO3 version 6.2.0 up to 7.99.99. When **TYPO3 7.2** was released, I found
out, that the YubiKey extension did'nt work any more with the new TYPO3 version, because the **TYPO3 backend login** has
been [rewritten](https://forge.typo3.org/issues/66431). I debugged into the issue and found out, that the new backend
login did not respect the template set in **$TBE\_STYLES\['htmlTemplates'\]\[$tmplPath\]**.

It seemed I was not the only one having this problem, as another user
already [reported](https://forge.typo3.org/issues/66669) the same issue on forge. So with TYPO3 7.2+, you can't use
$TBE\_STYLES\['htmlTemplates'\]\[$tmplPath\] to add own fields to the backend login form, since it is not supported any
more.

In this article I describe, how the new backend login form API is integrated in the **OpenID extension** and I also
describe, how I **extended the username/passwors login provider** to add an additional field to the default backend
login form.

### The new login form API in TYPO3 7.3

In order to get the missing functionality of extending the backend login form back, the TYPO3 core developers added
the **backend login form API** and also did a lot of **great work** refactoring the backend login. The backend login
form API uses **login providers** to enable extension developers to add an own backend login form for e.g. an own **
authentication service**. As an example, lets have a look at how the **OpenID authentication service** is integrated.

The OpenID authentication service registers a **new login provider** by adding the following code in the file
ext\_localconf.php

New login providers must add a new key to the array $GLOBALS\['TYPO3\_CONF\_VARS'\]\['EXTCONF'\]\['backend'
\]\['loginProviders'\]. The new key must be an **unix timestamp** (unique for all login providers). The following array
has the following keys:

* provider: The name of the login provider
* sorting: Defines the sorting order in case of there are multiple login providers
* icon-class: A class which defines the icon for the login provider
* label: the label for the login provider

Having the OpenID authentication service installed, the login provider of the OpenID extension adds a link to the
backend login (see screenshot below).

![TYPO3 7.3 backend login form showing the OpenID link](/assets/images/2015-06-16/image1.png)

When you follow the "**Login with OpenID**" link, you come to an own login form showing the OpenID field (screenshot
below).

![TYPO3 7.3 OpenID login form](/assets/images/2015-06-16/image2.png)

From there, you have the possibility to **switch back to the username and password** login. This is, because also the
username and password login has an own **login provider**.

Lets have a look at the contents of the login provider class.

As you can see, there is not much magic there. The **OpenidLoginProvider** implements the **LoginProviderInterface** and
has only one method (render), which is called in the LoginController of the TYPO3 backend login form. In case of the
OpenID extension, you see that **setTemplatePathAndFilename** is used to set the **Fluid template** for the OpenID login
form.

The template is kept really simple. It uses the layout "**Login**" (this is required) and has a section called 
"**loginFormFields**" (also required), which contains the OpenID login field.

### Integration of the backend login form API to the YubiKey extension

For the YubiKey extension, I could have used the same technique as the OpenID extension does, but since some TYPO3
installations may _require_ each user to use YubiKey two-factor authentication, it would be not very user-friendly to
let a user **manually switch** to the YubiKey backend login provider when opening the TYPO3 backend login form.

In order to add the YubiKey field to the backend login, I **extended the existing username and password login provider**
with my own login form. In the ext\_localconf.php, I added the following code.

Next I created the YubiKey login provider, which calls the parent login provider (username and password) and sets the
path to the Fluid template which contains the both the username and password fields and the YubiKey login field.

Note, that I _do not implement_ the LoginProviderInterface, but **extend** the **UsernamePasswordLoginProvider**.

Finally, after the YubiKey extension is installed and activated, the TYPO3 backend login now again has an additional
field for the YubiKey two factor authentication (screenshot below)

![TYPO3 7.3 backend login with the additional YubiKey field](/assets/images/2015-06-16/image3.png)

As TYPO3 7.3 has been released yesterday, extension authors ,who created own authentication providers for the TYPO3
backend, can now start to implement the new backend login API to their extensions.

The latest version 1.1.0 of the TYPO3 YubiKey extension uses the new backend login form API and is compatible with TYPO3
6.2 and TYPO3 7.x (except 7.2)