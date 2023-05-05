---
layout: post
title: How to create a custom CKEditor 5 plugin for TYPO3 12.4
date: '2023-05-05T13:00:00.000+02:00'
author: Torben Hansen
tags:
- TYPO3 12.4
- CKEditor 5
- plugin
modified_time: '2023-05-05T13:00:00.000+02:00'
permalink: /2023/05/2023-05-05-create-a-custom-ckeditor5-plugin-for-typo3-12.html
---

CKEditor is a popular WYSIWYG (What You See Is What You Get) editor used for creating rich-text content on the web. 
For many years, CKEditor 4 has been the go-to solution for content editors and developers alike. However, with the 
ever-evolving web technologies, CKEditor 4 has now been deprecated and replaced with the newer version, CKEditor 5.

Many content management systems (CMS) have already started to adopt CKEditor 5, and TYPO3 is no exception. With TYPO3 
12.4 LTS, CKEditor 4 has been replaced with CKEditor 5, providing TYPO3 users with a better editing experience and 
access to new features and improvements. In this way, TYPO3 stays ahead of the curve by incorporating the latest 
technology into its core.

One of the great advantages of TYPO3 is its extensibility. In case of CKEditor in TYPO3, it is possible to dynamically 
include custom plugins, allowing you to extend the functionality of the editor to meet your specific needs. 

In this article, I will show how to add a custom timestamp plugin (as shown in the TYPO3 
[documentation](https://docs.typo3.org/c/typo3/cms-core/main/en-us/Changelog/12.0/Breaking-96874-CKEditor-relatedPluginsAndConfiguration.html)) 
to TYPO3 12.4 LTS

### Prerequisites

First of all, a custom sitepackage is required, where the configuration for CKEditor is stored. In my example,
I use `sitepackage` as extension key for the sitepackage. 

In `ext_localconf.php`, the path to the custom CKEditor configuration is defined. 

{% highlight php %}
$GLOBALS['TYPO3_CONF_VARS']['RTE']['Presets']['default'] = 'EXT:sitepackage/Configuration/RTE/Default.yaml';
{% endhighlight %}

The file `Configuration/RTE/Default.yaml` contains the default CKEditor 5 configuration for TYPO3, which has been 
copied from `ext:rte_ckeditor/Configuration/RTE/Default.yaml`.

### 1. Create the CKEditor 5 plugin

The TYPO3 [documentation](https://docs.typo3.org/c/typo3/cms-core/main/en-us/Changelog/12.0/Breaking-96874-CKEditor-relatedPluginsAndConfiguration.html)
contains an example for a custom timestamp plugin. It is important to know, that the shown code is TypeScript
(e.g `readonly` attribute for `pluginName` constant or `: void` as return type for the `init()` function). The code
will not directly run as CKEditor 5 plugin and either needs to be **compiled** or **adapted**. For simplicity
reasons, the code in this example is just adapted as shown below.

The file `Resources/Public/JavaScript/Ckeditor/timestamp-plugin.js` contains the following ES6 JavaScript code.

{% highlight javascript %}
import {Core, UI} from '@typo3/ckeditor5-bundle.js';

export default class Timestamp extends Core.Plugin {
  static pluginName = 'Timestamp';

  init() {
    const editor = this.editor;

    // The button must be registered among the UI components of the editor
    // to be displayed in the toolbar.
    editor.ui.componentFactory.add(Timestamp.pluginName, () => {
      // The button will be an instance of ButtonView.
      const button = new UI.ButtonView();

      button.set( {
        label: 'Timestamp',
        withText: true
      } );

      //Execute a callback function when the button is clicked
      button.on('execute', () => {
        const now = new Date();

        //Change the model using the model writer
        editor.model.change(writer => {

          //Insert the text at the user's current position
          editor.model.insertContent(writer.createText( now.toString()));
        });
      });

      return button;
    });
  }
}
{% endhighlight %}

It is highly recommended to read [ES6 in the TYPO3 Backend](https://docs.typo3.org/m/typo3/reference-coreapi/main/en-us/ApiOverview/Backend/JavaScript/ES6/Index.html#id1) 
in order to understand how ES6 is used in TYPO3 and how to migrate from RequireJS. 

### 2. Register the ES6 JavaScript

Another new feature in TYPO3 12.4 is the global usage of JavaScript ES6 modules in TYPO3 Backend. According to the 
[documentation](https://docs.typo3.org/m/typo3/reference-coreapi/main/en-us/ApiOverview/Backend/JavaScript/ES6/Index.html), 
custom JavaScript ES6 modules must be registered. This has to be done in the `EXT:my_extension/Configuration/JavaScriptModules.php`.

In this example, the file `EXT:sitepackage/Configuration/JavaScriptModules.php` contains the following code:

{% highlight php %}
<?php

return [
    'dependencies' => ['backend'],
    'tags' => [
        'backend.form',
    ],
    'imports' => [
        '@derhansen/sitepackage/timestamp-plugin.js' => 'EXT:sitepackage/Resources/Public/JavaScript/Ckeditor/timestamp-plugin.js',
    ],
];
{% endhighlight %}

In this file, an importmap for `@derhansen/sitepackage/timestamp-plugin.js` is defined and the JavaScript file with the 
CKEditor plugin is referenced.

### 3. Extend TYPO3 CKEditor configuration

Finally the TYPO3 CKEditor configuration needs to be extended, so the new plugin is registered using `importModules` 
and the plugin button is added to the `toobar` items. The following extract from the CKEditor configuration
shows the important parts.

```
editor:
  config:
    importModules:
      - '@derhansen/sitepackage/timestamp-plugin.js'
    toolbar:
      items:
        - style
        - heading
        # grouping separator
        - '|'
        - bold
        - italic
        - subscript
        - superscript
        - softhyphen
        - '|'
        - bulletedList
        - numberedList
        - blockQuote
        - alignment
        - '|'
        - findAndReplace
        - link
        - '|'
        - removeFormat
        - undo
        - redo
        - '|'
        - insertTable
        - '|'
        - specialCharacters
        - horizontalLine
        - sourceEditing
        - '|'
        - timestamp
```

### 4. Using the plugin

After the plugin has been added to the CKEditor configuration, the TYPO3 cache must be cleared. After reloading the
TYPO3 backend, the CKEditor now contains the new plugin as shown below.

![CKEditor 5 in TYPO3](/assets/images/2023-05-05/typo3-ckeditor5-plugin.png)

When the "Timestamp" button is clicked, the current date is inserted into the editor content.

