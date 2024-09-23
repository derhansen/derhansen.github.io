---
layout: post
title: 'Extending an existing extbase backend module with a custom action in TYPO3 v12+'
date: '2024-09-23T08:30.000+02:00'
author: Torben Hansen
tags:
- TYPO3
- extend backend module
- extbase
- action
modified_time: '2024-09-23T08:30.000+02:00'
permalink: /2024/09/extending-an-existing-extbase-backend-module-with-a-custom-action-in-typo3.html
---

One part I really like about TYPO3 is its extendability. The TYPO3 core and its APIs contain many PSR-14 events 
and hooks, which developers can use to customize or extend the functionality of the system. My TYPO3 extension 
[sf_event_mgt](https://extensions.typo3.org/extension/sf_event_mgt/) has an extbase based backend module, which 
allows editors to manage events and event registrations. It is for example possible to export a CSV list of 
participants or to notify the participants of an event by email. Sometimes, the included features of the backend 
module however do not cover all customer demands. In such situations, I usually **extend** the existing backend 
module with the needed features, so the customer do not have to use 2 backend modules to manage events.

<div class="alert-warning">
    <h3>🔥🔥🔥 Never modify code in a public extension for your project 🔥🔥🔥</h3>
    You should never add custom code directly in local instances of public TYPO3 extensions, since you will end up  
    in a state, where it will be <strong>hard or impossible to update</strong> the extension. This is really no fun  
    in terms of TYPO3 <strong>major updates</strong> and also pretty bad in case of a <strong>security issue</strong> 
    in the public extension, where you then manually have to apply the security patch.
</div>

### Adding a new action to a backend module of an existing extension

In the following step-by-step guide, I will show how extend the existing extbase backend module of an extension 
with a custom action. My TYPO3 extension sf_event_mgt will be the extension where the backend module is extended.

#### 1. Create a new plain extension

First step is to create a new extension, which will hold all code for the required functionality. Ensure, that the 
new extension depends on the extension you extend. To do so, add the "main" extension as dependency in 
`composer.json` file (TYPO3 composer mode) or `ext_emconf.php` file (TYPO3 classic mode). This will ensure, that the 
extending extension is loaded **after** the extension you extend. In this example, I will use `sf_event_mgt_extend` as 
extensions name.

#### 2. Define template overrides for backend module

To override templates of a TYPO3 backend module, it is required to define template overrides. Since TYPO3 12, this 
is achieved using TSconfig (see https://docs.typo3.org/c/typo3/cms-core/main/en-us/Changelog/12.0/Feature-96812-OverrideBackendTemplatesWithTSconfig.html).

```
templates.derhansen/sf_event_mgt.20 = derhansen/sf_event_mgt_extend:Resources/Private
```

The backend module of sf_event_mgt will then also look for template overrides in sf_event_mgt_extend

#### 3. XCLASS the controller of the original backend module

Although XCLASSing is considered risky, it as a nice possiblity to add functionality to an existing class. I 
personally only use XCLASS to add functionality and never to overwrite something in the XCLASSed class.

In `ext_localconf.php`, I define the new class for the XLCASSed class.

```
$GLOBALS['TYPO3_CONF_VARS']['SYS']['Objects'][\DERHANSEN\SfEventMgt\Controller\AdministrationController::class] = [
    'className' => \Derhansen\SfEventMgtExtend\Controller\AdministrationController::class
];
```

And in `Classes/Controller/AdministrationController.php`, I extend the original controller with the custom action as 
shown below.

```
<?php

declare(strict_types=1);

namespace Derhansen\SfEventMgtExtend\Controller;

use Psr\Http\Message\ResponseInterface;

class MyAdministrationController extends \DERHANSEN\SfEventMgt\Controller\AdministrationController
{
    public function doSomethingAction(): ResponseInterface
    {
        $variables = [
            'myVariable' => 'Variable content',
        ];

        return $this->initModuleTemplateAndReturnResponse('Administration/DoSomething', $variables);
    }
}
```

Note, that depending on the extension you extend, it may be, that you have to manually constuct the module template 
response.

#### 3. Add event listener to register custom action

Next, you have to make the original backend module aware of the new action. To do so, an event lister for the 
`BeforeModuleCreationEvent` has to be used, where the `controllerActions` array of the original backend module 
controller is extended.

Register the event listener in the `Configuration/Services.yaml` file of the extending extension as shown below:

```
Derhansen\SfEventMgtExtend\EventListener\ExtendAdministrationModule:
  tags:
    - name: event.listener
      identifier: 'sf-event-extend/backend/extend-administration-module'
```

**Side note:** In TYPO3 v13, you can also use the [AsEventListener](https://api.typo3.org/main/classes/TYPO3-CMS-Core-Attribute-AsEventListener.html) 
PHP attribute and skip the registration in `Configuration/Services.yaml`.

Next, implement the event listener in `Classes/EventListener/ExtendAdministrationModule.php` as shown below:

```
<?php

declare(strict_types=1);

namespace Derhansen\SfEventExtend\EventListener;

use DERHANSEN\SfEventMgt\Controller\AdministrationController;
use TYPO3\CMS\Backend\Module\BeforeModuleCreationEvent;

class ExtendAdministrationModule
{
    public function __invoke(BeforeModuleCreationEvent $event): void
    {
        $configuration = $event->getConfiguration();
        if (($configuration['extensionName'] ?? '') === 'SfEventMgt') {
            $configuration['controllerActions'][AdministrationController::class][] = 'doSomething';
            $event->setConfiguration($configuration);
        }
    }
}
```

#### 4. Create fluid template for new action

Since the new action is pretty simple, I just add a fluid template for the action by adding the file 
`Resources/Private/Templates/Administration/DoSomething.html`.

#### 5. Use new action in backend module templates

The new action is now ready to be used in templates. As an example, I extend the `ListItem.html` partial of the 
original extension by defining a template override in the extending extension. I add the file 
`Resources/Private/Partials/Administration/ListItem.html` and copy the content of the original file of the main 
extension to it. Next, I add the following new HTML code to the extended partial.

```
<f:link.action action="doSomething" class="btn btn-default btn-sm" title="Do something">
    <core:icon identifier="actions-list-alternative" size="small" />
</f:link.action>
```

Finally, the new action can be used in the existing backend module of sf_event_mgt. 

