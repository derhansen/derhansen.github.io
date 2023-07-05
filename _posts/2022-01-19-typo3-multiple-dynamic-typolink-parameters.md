---
layout: post
title: TYPO3 - Multiple dynamic parameters for a typolink using a custom userFunc
date: '2022-01-19T15:53:00.014+01:00'
author: Torben Hansen
tags:
- userfunc
- TYPO3
- typolink
modified_time: '2023-07-05T22:40:00.000+01:00'
blogger_id: tag:blogger.com,1999:blog-6517038209122183182.post-6637893069431677698
blogger_orig_url: http://www.derhansen.de/2022/01/typo3-multiple-dynamic-typolink-parameters.html
permalink: /2022/01/typo3-multiple-dynamic-typolink-parameters.html
---

I often use the TYPO3 linkHandler to enable the possibility for editors to create direct links to records from within
the CKEditor in TYPO3 backend. This is
all [well documented](https://docs.typo3.org/m/typo3/reference-coreapi/11.5/en-us/ApiOverview/LinkBrowser/Linkhandler/Index.html)
and easy to configure using the RecordLinkHandler, as long as the resulting link only contains one dynamic parameter.
But sometimes it may be required to have **multiple dynamic parameters** for the resulting link. In this case you may
need to create a userFunc for the typolink function in order to create a custom configuration which uses multiple
dynamic parameters.

#### Requirement

Let us assume, you have an event which has multiple event registrations. Registrations are listed in the detail view of
an event and each registration is shown as an accordion item with a unique ID in markup. Now you want to create a link
to an event and set a link anchor to a specific registration. The resulting URL should be as shown below:

https://www.cool-events.tld/events/my-first-event#registration-1

Calling the URL will open the event detail page and scroll down the the HTML element with the ID "registration-1".

_**Note:** This is just an example, which also can be achieved without a custom userFunc. Goal of this article is to demonstrate how to use a userFunc for typolink._

#### Solution

In order to archive the requirement, first a linkHandler Page TSConfig must be created as shown below:

{% highlight php %}
TCEMAIN.linkHandler {
    event {
        handler = TYPO3\CMS\Recordlist\LinkHandler\RecordLinkHandler
        label = Event Registration
        configuration {
            table = tx_sfeventmgt_domain_model_registration
        }
    }
}
{% endhighlight %}

Next, the TypoScript for the link generation is added. 

{% highlight php %}
config {
    recordLinks {
        registration {
            typolink {
                parameter = 1
                userFunc = DERHANSEN\SfEventMgt\UserFunc\TypoLink->createEventLink
                userFunc {
                    eventUid = TEXT
                    eventUid.data = field:event
                    registrationUid = TEXT
                    registrationUid.data = field:uid
                }
            }
        }
    }
}
{% endhighlight %}

Finally, a custom userFunc needs to be created which renders the A-tag for the link.

{% highlight php %}
<?php

declare(strict_types=1);

namespace DERHANSEN\SfEventMgt\UserFunc;

use TYPO3\CMS\Frontend\ContentObject\ContentObjectRenderer;

class TypoLink
{
    private const EVENT_DETAILPID = 22;

    // TYPO3 < 12.4
    //public $cObj;

    // TYPO3 12.4+ only
    protected $cObj;

    public function setContentObjectRenderer(ContentObjectRenderer $cObj): void
    {
        $this->cObj = $cObj;
    }

    public function createEventLink(array $content, array $config): string
    {
        $eventUid = $this->cObj->cObjGetSingle($config['eventUid'], $config['eventUid.']);
        $registrationUid = $this->cObj->cObjGetSingle($config['registrationUid'], $config['registrationUid.']);

        // Link parameters (can also contain multiple dynamic parameters)
        $parameters = [
            'tx_sfeventmgt_pieventdetail' => [
                'controller' => 'Event',
                'action' => 'detail',
                'event' => $eventUid,
            ]
        ];

        $link = $this->cObj->typoLink($this->cObj->lastTypoLinkResult->getLinkText(), [
            'parameter' => self::EVENT_DETAILPID,
            'additionalParams' => '&' . http_build_query($parameters),
            'section' => 'registration-' . $registrationUid,
            'returnLast' => 'url',
        ]);

        return '<a href="' . $link . '">';
    }
}
{% endhighlight %}

The most important part is, that the custom userFunc must only return the **opening A-tag**. In the userFunc, it is
basically possible to construct the resulting link however you want. In the example above, 2 dynamic parameters are 
used in the function (_$eventUid_ and _$registrationUid_). It is of course also possible to e.g. do dynamic **database 
lookups** in the function to fetch other dynamic parameters required for link construction.

