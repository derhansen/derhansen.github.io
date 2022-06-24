---
layout: post
title: Using TYPO3 FluidEmail in CLI context
date: '2022-06-23T10:07:00.003+02:00'
author: Torben Hansen
tags:
- TYPO3
- FluidEmail
- StandaloneView
- CLI
- Symfony Console
modified_time: '2022-06-23T10:17:21.663+02:00'
blogger_id: tag:blogger.com,1999:blog-6517038209122183182.post-4609028560548194137
blogger_orig_url: http://www.derhansen.de/2022/06/using-typo3-fluidemail-in-cli-context.html
permalink: /2022/06/using-typo3-fluidemail-in-cli-context.html
---
Since TYPO3 10.4 it is possible to use `TYPO3\CMS\Core\Mail\FluidEmail` to send emails with body content rendered
by Fluid. This all works fine as long as you work in frontend or backend context, but when you use FluidEmail in CLI
context (e.g. in a symfony console command), you will run into some problems. Since no real server request object is
available, TYPO3 is not able to construct URIs using Fluid ViewHelpers nor will the Image ViewHelper render working
absolute image links. Especially the last problem (non working website logo in rendered FluidEmail default template)
motivated me to dig deeper into the topic.

Below follows a summary of things you should consider when working with FluidEmail (or Fluid StandaloneView) in CLI
context.

### All Fluid ViewHelpers in the namespace f:uri or f:link do not work by default

The ViewHelpers depend on either a TypoScriptFrontendController object or the global TYPO3_REQUEST variable being
available. Both are not in CLI

One workaround for this problem is to create an instance of `$GLOBALS['TYPO3_REQUEST']` manually in your symfony
console command as shown below:

{% highlight php %}
$site = GeneralUtility::makeInstance(SiteFinder::class)->getSiteByPageId(1);
$request = (new ServerRequest())
    ->withAttribute('applicationType', SystemEnvironmentBuilder::REQUESTTYPE_FE)
    ->withAttribute('site', $site);
$GLOBALS['TYPO3_REQUEST'] = $request;
{% endhighlight %}

Note, that the site object is fetched by the uid of page 1. This must be kept in mind, when you have a TYPO3 website
with multiple sites defined.

### Link created with f:link.action must always contain the extensionName
If the previous workaround is used, it is mandatory to set the "extensionName" argument in the `f:link.action` or
`f:uri.action` ViewHelper.

### Create links manually

An alternative to the previous example, where the $GLOBALS['TYPO3_REQUEST'] object is created manually, is to create
all links manually used in the FluidEmail template in the symfony console command. This can be done as shown below:

{% highlight php %}
$arguments = [
    'tx_sfeventmgt_pieventdetail' => [
        'action' => 'detail',
        'controller' => 'Event',
        'event' => 25,
    ],
];

$site = GeneralUtility::makeInstance(SiteFinder::class)->getSiteByPageId(1);
$eventLink = (string)$site->getRouter()->generateUri(22, $arguments);
{% endhighlight %}

Note again, that the site object is fetched by the uid of page 1. The variable $eventLink can then be passed to the
view and be used as href attribute for a-tags.

### Image ViewHelper creates broken URLs

When you try to render an image with an absolute path using the f:image ViewHelper, the resulting link will be broken.
Example:

{% highlight html %}
<f:image absolute="1" src="EXT:sf_event_mgt/Resources/Public/Icons/Extension.svg" width="100" />
{% endhighlight %}

In CLI context this will create an img tag as shown below:

{% highlight html %}
<img src="http://./typo3/sysext/core/bin/typo3conf/ext/sf_event_mgt/Resources/Public/Icons/Extension.svg" width="100" height="104" alt="">
{% endhighlight %}

In order to create a workaround for the problem, I found 2 different approaches.

**Workaround 1**

FluidEmail has the function setRequest() which allows to set a ServerRequest object for the view. 

{% highlight php %}
$site = GeneralUtility::makeInstance(SiteFinder::class)->getSiteByPageId(1);

$normalizedParams = new NormalizedParams(
    [
        'HTTP_HOST' => $site->getBase()->getHost(),
        'HTTPS' => $site->getBase()->getScheme() === 'https' ? 'on' : 'off',
    ],
    $systemConfiguration ?? $GLOBALS['TYPO3_CONF_VARS']['SYS'],
    '',
    ''
);

$request = (new ServerRequest())
    ->withAttribute('applicationType', SystemEnvironmentBuilder::REQUESTTYPE_FE)
    ->withAttribute('normalizedParams', $normalizedParams)
    ->withAttribute('site', $site);

$email->setRequest($request);
{% endhighlight %}

By assigning the request object to the view, the variable `{normalizedParams.siteUrl}` is now also available in CLI 
context resulting in the TYPO3 logo being shown again in the default email layout. 

**Workaround 2**

The `f:image` ViewHelper will create relative links to images when the argument "absolute" is not used. So basically just
the baseUri is missing. In the previous workaround `{normalizedParams.siteUrl}` was used to add the baseUri, but instead
it is also possible to assign the sites baseUri as Fluid variable as shown below:

{% highlight php %}
$site = GeneralUtility::makeInstance(SiteFinder::class)->getSiteByPageId(1);
$email->assign('baseUri', (string)$site->getBase());
{% endhighlight %}

In the template the full path to the image can now be created as following:

{% highlight html %}
<img src="{baseUri}{f:uri.image(src: 'EXT:sf_event_mgt/Resources/Public/Icons/Extension.svg', width: '100')}" width="100" />
{% endhighlight %}

### Ensure to set the correct application context in CLI

Whenever the site object is fetched using the SiteFinder, the resulting base is resolved respecting TYPO3s application
context. By default this is "production". If you e.g. have a testing or staging environment and use one of the described
workarounds, ensure to set the application context as environment variable as shown below:

{% highlight shell %}
export TYPO3_CONTEXT=Development
{% endhighlight %}
