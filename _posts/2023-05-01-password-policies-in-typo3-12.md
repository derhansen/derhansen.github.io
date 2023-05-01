---
layout: post
title: How to extend Password Policy validation in TYPO3 12.4
date: '2023-05-01T12:15:00.000+02:00'
author: Torben Hansen
tags:
- TYPO3 12.4
- password
- validation
- security
modified_time: '2023-05-01T13:15:00.000+02:00'
permalink: /2023/05/2023-05-01-how-to-extend-password-policy-validation-in-typo3-12.html
---

With TYPO3 version 12, the new global password policies feature has been introduced (see forge issue 
[97387](https://forge.typo3.org/issues/97387). It allows to define one or multiple password validators to
be used in different scopes. The TYPO3 core includes 2 password validators, which are configured in the password
policy named `default`. This password policy is used in frontend and backend scope.

The official [documentation](https://docs.typo3.org/m/typo3/reference-coreapi/main/en-us/ApiOverview/PasswordPolicies/Index.html#password-policies) 
about password policies shows, which TYPO3 core password validators are available and how they can be configured.

While the default password policy can be effective in improving security, it may not always meet the specific 
requirements of customers. For example, some customers may require to ensure, that a user password has not been
part of a known data breach. Other customers may require, that a user password must not include the username. 
In case you meet such requirements, it is possible to add custom password validators to a password policy in TYPO3.

### TYPO3 extension add_pwd_policy

While implementing the password policy feature into TYPO3, I also developed a TYPO3 extension to verify, that the 
Password Policy feature is as flexible as planned. This extension is now available as "add_pwd_policy" on
[GitHub](https://github.com/derhansen/add_pwd_policy/), 
[TYPO3 TER](https://extensions.typo3.org/extension/add_pwd_policy) and 
[packagist](https://packagist.org/packages/derhansen/add_pwd_policy) and contains custom password validators, which
directly can be used in a TYPO3 password policy. The extension currently contains the following password 
validators:

#### Pwned Password
This validator ensures, that the given password is not part of a known data breach on [haveibeenpwned.com](https://haveibeenpwned.com)

#### Does not contain username
This validator ensures, that the given password does not contain the users `username`.

#### Password deny list
This validator ensures, that the given password is not part of a configurable list of denied passwords.

The code of the [password validators](https://github.com/derhansen/add_pwd_policy/tree/main/Classes/PasswordPolicy/Validator) included 
in the extension can also be used as examples on how to create custom password validators for TYPO3. 

Additionally, the extension shows, how the TYPO3 core `TYPO3\CMS\Core\PasswordPolicy\Event\EnrichPasswordValidationContextDataEvent`
PSR-14 event can be used to enrich context data used for password validation. As an example, the included 
[Event Listener](https://github.com/derhansen/add_pwd_policy/blob/main/Classes/EventListener/EnrichContextData.php) adds 
the users email-address to the password validation context data object. 

### PRs welcome

If you want to contribute a custom password validator that meets your specific requirements and which you want to share
with the TYPO3 community, you are welcome to create a pull request on GitHub. 



