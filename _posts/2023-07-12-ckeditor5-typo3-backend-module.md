---
layout: post
title: How to use CKEditor 4 and 5 in a TYPO3 backend module
date: '2023-07-13T08:12:00.000+02:00'
author: Torben Hansen
tags:
- typo3
- backend module
- ckeditor
modified_time: '2023-07-13T08:12:00.000+02:00'
permalink: /2023/07/2023-07-13-ckeditor-4-and-5-typo3-backend-module.html
---

In TYPO3 backend modules, it can be useful to provide a CKEditor instance, for example when creating 
content like dynamic HTML emails. The user then has the possibility to enrich the email content with 
basic formatting like making text appear in bold, italic or underline or even add links.

I sometimes use this possibility on websites, where my TYPO3 extension sf_event_mgt is installed and 
where the editor requires more flexibility for email notifications sent through the backend module of the 
extension.  

Adding a CKEditor instance to a TYPO3 backend module is however different depending on which TYPO3 
version is used. The examples below show, how this works with TYPO3 12.4 and CKEditor 5 and with 
TYPO3 <= 11.5 and CKEditor 4.

### Adding a CKEditor 5 instance to a textfield

For **TYPO3 12.4**, it is required to add a custom ES6 module in order to enable CKEditor 5 for a 
textfield in a backend module.

First it is required to import required modules in the file `Configuration/JavaScriptModules.php`
as shown below:

{% highlight php %}
<?php

return [
    'dependencies' => ['backend'],
    'tags' => [
        'backend.form',
    ],
    'imports' => [
        '@typo3/rte-ckeditor/' => 'EXT:rte_ckeditor/Resources/Public/JavaScript/',
        '@typo3/ckeditor5-bundle.js' => 'EXT:rte_ckeditor/Resources/Public/Contrib/ckeditor5-bundle.js',
        '@mynamespace/my-extension-key/' => 'EXT:my_extension-key/Resources/Public/JavaScript/Backend/',
    ],
];
{% endhighlight %}

Next, a custom ES6 module with a very basic CKEditor 5 configuration is added to 
`Resources/Public/JavaScript/Backend/my-module.js` as shown below:

{% highlight php %}
import { CKEditor5 } from '@typo3/ckeditor5-bundle.js';

class NotificationModule {
    constructor() {
        let target = document.getElementById('myTextfieldId');

        const config = {
            toolbar: [ 'bold', 'italic', '|', 'bulletedList', 'numberedList', '|', 'sourceEditing', '|', 'link' ],
        }

        CKEditor5.create(target, config);
    }
}

export default new NotificationModule();
{% endhighlight %}

Finally, the module needs to be loaded for the backend view, where the textfield is present. This can either 
be done in the controller or simply by using the `f:be.pageRenderer` ViewHelper in the view Template as shown below:

{% highlight php %}
<f:be.pageRenderer includeJavaScriptModules="{0: '@mynamespace/my-extension-key/my-module.js'}" />
{% endhighlight %}

For CKEditor 5, it is not possible to define the hight of the editor instance through configuration. Therefore
it is recommended to include a custom CSS file, which defines the height of the editor field.

#### Screenshot

![CKEditor 5 instance in TYPO3 backend module](/assets/images/2023-07-13/ckeditor-5-backend-module.png)

### Adding a CKEditor 4 instance to a textfield

For **TYPO3 11.5 or less**, it is required to add a custom Require JS module in 
`Resources/Public/JavaScript/Backend/my-module.js` to enable CKEditor 4 for a textfield in a backend 
module as shown in the example below:

{% highlight php %}
define(['TYPO3/CMS/RteCkeditor/Contrib/Ckeditor'], function($) {
    'use strict';

    // Initialize CKEditor for field additionalMessage
    var config = {
        stylesSet: [],
        contentsCss: [],
        customConfig: '',
        height: 150,
        toolbarGroups: [
            {
                'name': 'basicstyles',
                'groups': ['basicstyles']
            },
            {
                'name': 'paragraph',
                'groups': ['list', 'blocks']
            },
            {
                'name': 'document',
                'groups': ['mode']
            },
            {
                'name': 'links',
                'groups': ['links']
            }
        ],
        removeButtons: 'Underline,Strike,Subscript,Superscript,Anchor,Styles,Specialchar,Blockquote',
        linkShowTargetTab: false
    }

    CKEDITOR.replace('myTextfieldId', config)
});
{% endhighlight %}

This module defines a very basic CKEditor 4 for the field `myTextfieldId`. The module is then either included through
the backend module controller or simply using a Fluid ViewHelper in the view template as shown below:

{% highlight php %}
<f:be.pageRenderer includeRequireJsModules="{0: 'TYPO3/CMS/MyExtensionKey/Backend/MyRequireJsModule'}" />
{% endhighlight %}

#### Screenshot

![CKEditor 4 instance in TYPO3 backend module](/assets/images/2023-07-13/ckeditor-4-backend-module.png)
