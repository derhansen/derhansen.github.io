---
layout: post
title: 'How to use flux and container extension in one TYPO3 system'
date: '2025-10-24T14:20.000+02:00'
author: Torben Hansen
tags:
- flux
- container
- TYPO3
modified_time: '2025-10-24T14:20.000+02:00'
permalink: /2025/10/how-to-use-flux-and-container-extension-in-one-typo3-system.html
---

TYPO3 is well known for its flexibility, extensibility, and sustainability. It is not uncommon to find TYPO3 
installations that were originally built many years ago and which are still actively maintained and updated today. 
In some of those older setups, site layouts and content elements were implemented using the wellknown 
[flux](https://extensions.typo3.org/extension/flux) extension. 

As TYPO3 has evolved, so have the also available technologies. Today TYPO3 integrators usually use TYPO3 core
content elements, [content_blocks](https://extensions.typo3.org/extension/content_blocks) and 
[container](https://extensions.typo3.org/extension/container) to provide website layouts and content.

## Why use flux and container extension in one TYPO3 system?

In longterm running **multi-site TYPO3 systems**, it is not unusual to encounter different technological approaches
across individual sites. Some sites may rely on **flux**, while newer ones might use **container**. However, using both
extensions within the same TYPO3 system is not possible out of the box, as they both override the `itemsProcFunc`
for the `colPos` field of the `tt_content` table. This usually results in problems when adding new content elements
to structured content elements which have **content columns** as shown in the image below.

![Missing label for colPos field](/assets/images/2025-10-24/typo3-colpos-missing-label.png)

The issue tracker of the container extension has several closed issues ([example](https://github.com/b13/container/issues/629)) 
related to this problem. I, however, could not find any issue with a solution on how to solve this problem on a site 
basis in one TYPO3 system.

## Creating an own itemsProcFunc to handle both extensions

In [this](https://github.com/b13/container/issues/115#issuecomment-780393942) issue in the issue tracker of the 
container extension I found a hint on how to resolve the problem. Other than shown, I just need to ensure to
apply the `itemsProcFunc` of each extension depending on the **current site**.

**Important:** First it has to be ensured, that my custom `itemsProcFunc` is loaded **after** the flux and container 
extension is loaded. To do so, I added **dependencies** to both extensions in the `composer.json` of my sitepackage 
extension.

Next, I registerted a custom `itemsProcFunc` in `ext:sitepackage/Configuration/TCA/Overrides/tt_content.php` as shown below:

```php
$GLOBALS['TCA']['tt_content']['columns']['colPos']['config']['itemsProcFunc'] = \Vender\Extension\UserFunc\ItemProcFunc::class . '->colPos';
```

Next, I created a new class `ItemProcFunc` in `ext:sitepackage/Classes/UserFunc/ItemProcFunc.php` as shown below:

```php
<?php

namespace Vendor\Extension\UserFunc;

use FluidTYPO3\Flux\Integration\Overrides\BackendLayoutView;
use TYPO3\CMS\Core\Site\Entity\Site;
use TYPO3\CMS\Core\Utility\GeneralUtility;

class ItemProcFunc
{
    public function colPos(array $parameters): void
    {
        $fluxSites = ['site1', 'site2', 'site3'];

        /** @var Site|null $site */
        $site = $parameters['site'] ?? null;
        if (!$site) {
            return;
        }

        if (in_array($site->getIdentifier(), $fluxSites, true)) {
            // ItemProcFunc for ext:flux sites
            $itemsProcFunc = GeneralUtility::makeInstance(BackendLayoutView::class);
            $itemsProcFunc->colPosListItemProcFunc($parameters);
        } else {
            // ItemProcFunc for ext:container sites
            $itemsProcFunc = GeneralUtility::makeInstance(\B13\Container\Tca\ItemProcFunc::class);
            $itemsProcFunc->colPos($parameters);
        }
    }
}
```

So basically, the function just checks if the current site is part of the list of flux sites and calls the 
appropriate `itemsProcFunc` of the corresponding extension.

Having the custom `itemsProcFunc` in place, I can now use both extensions in one TYPO3 system for individual sites.