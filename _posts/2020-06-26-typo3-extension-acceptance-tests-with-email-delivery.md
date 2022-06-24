---
layout: post
title: Testing email delivery of a TYPO3 extension with Codeception, MailHog and GitHub
  Actions
date: '2020-06-26T14:20:00.000+02:00'
author: Torben Hansen
tags:
- MailHog
- TYPO3 CMS
- GitHub Actions
- Acceptance Testing
modified_time: '2020-06-26T14:20:19.949+02:00'
blogger_id: tag:blogger.com,1999:blog-6517038209122183182.post-364573512324616464
blogger_orig_url: http://www.derhansen.de/2020/06/typo3-extension-acceptance-tests-with-email-delivery.html
permalink: /2020/06/typo3-extension-acceptance-tests-with-email-delivery.html
---

Some weeks ago I published
my [blogpost](https://www.derhansen.de/2020/05/typo3-extension-testing-with-github-actions.html) about how to create a
GitHub Actions build pipeline for a TYPO3 Extension that executes Unit-, Functional- and Acceptance tests. The extension
tested in that blogpost was only a simple demo extension and for me this was a preparation to finally migrate the CI
pipeline for my TYPO3 extension [sf\_event\_mgt](https://extensions.typo3.org/extension/sf_event_mgt/) to GitHub
Actions.

The extension comes with lots of unit and functional tests, which are automatically executed on each commit. One missing
piece in the puzzle was the automatic execution of my **Acceptance Tests,** which are based
on [Codeception](http://codeception.com/) and additionally require [MailHog](https://github.com/mailhog/MailHog) in
order to test if emails are sent by the extension and if the email content is as expected.

The concept of testing emails in Acceptance Tests using Codeception is really simple. You have to add the composer
package [ericmartel/codeception-email-mailhog](https://packagist.org/packages/ericmartel/codeception-email-mailhog) to
your dev dependencies and then you are ready to test emails as shown in the abstract of one of my tests below:

{% highlight php %}
$I->fetchEmails();  
$I->haveUnreadEmails();  
$I->haveNumberOfUnreadEmails(2);  
$I->openNextUnreadEmail();  
$I->seeInOpenedEmailSubject('New unconfirmed registration for event "Event (reg, cat1) ' . $this->lang . '"');  
$I->seeInOpenedEmailRecipients('admin@sfeventmgt.local');  
$I->openNextUnreadEmail();  
$I->seeInOpenedEmailSubject('Your registration for event "Event (reg, cat1) ' . $this->lang . '"');  
$I->seeInOpenedEmailRecipients('johndoe@sfeventmgt.local');
{% endhighlight %}

It is also possible to check the email body for various content like I do in other parts of
my [testsuite](https://github.com/derhansen/sf_event_mgt/tree/develop/Tests/Acceptance).

GitHub Actions supports [docker based service containers](https://help.github.com/en/actions/reference/workflow-syntax-for-github-actions#jobsjob_idservices)
and MailHog is also available as docker container, so in order to execute my Acceptance testsuite I added MailHog as
service container to my CI setup as shown below:

{% highlight php %}
jobs:
  build:
    runs-on: ubuntu-18.04
    services:
      mailhog:
      image: mailhog/mailhog
      ports:
        - 1025:1025
        - 8025:8025
{% endhighlight %}

Having the MailHog container in place, the execution of the Acceptance Tests works like a charm.

Since the Acceptance Tests also cover tests of a plugin that is only accessible by logged in frontend users, the TYPO3
website for Acceptance Tests includes a special configured page with ext:felogin for this scenario. It turned out, that
those tests failed on GitHub actions, since Argon2i was not available on the testing runner for whatever reasons. In
order to resolve this problem, I configured the TYPO3 website to use BcryptPasswordHash instead of Argon2i which is ok
for me, since strong password hashes are not really required in this scenario.

The [GitHub actions YAML file](https://github.com/derhansen/sf_event_mgt/blob/develop/.github/workflows/ci.yml) is
currently available in the development branch of my extension.

The CI results including a downloadable Codeception HTML report for all acceptance tests is available for each build as
shown in this
example: [https://github.com/derhansen/sf\_event\_mgt/actions/runs/142799855](https://github.com/derhansen/sf_event_mgt/actions/runs/142799855)