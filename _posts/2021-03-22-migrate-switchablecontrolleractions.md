---
layout: post
title: How to migrate switchableControllerActions in a TYPO3 Extbase extension to
  single plugins
date: '2021-03-22T11:52:00.001+01:00'
author: Torben Hansen
tags:
- switchableControllerAction
- "#89463"
- TYPO3 11
- migrate
modified_time: '2021-03-22T11:57:36.184+01:00'
thumbnail: https://1.bp.blogspot.com/-LmDlVtWjpiY/YFWUdMw30pI/AAAAAAAAraA/BtCskzqYxycTUoiNbEDgtktJtl9wvARpQCLcBGAsYHQ/s72-c/Bildschirmfoto%2B2021-03-20%2Bum%2B07.21.17.png
blogger_id: tag:blogger.com,1999:blog-6517038209122183182.post-4619455578044088027
blogger_orig_url: http://www.derhansen.de/2021/03/migrate-switchablecontrolleractions.html
permalink: /2021/03/migrate-switchablecontrolleractions.html
---

**TL;DR** \- I created this [TYPO3 update wizard](https://gist.github.com/derhansen/4524495ccfef9335c96d6d535bad7324)
which migrates plugins and Extbase plugin settings for each former switchable controller actions configuration entry.

Since switchableControllerActions in Extbase plugins have
been [deprecated](https://docs.typo3.org/c/typo3/cms-core/master/en-us/Changelog/10.3/Deprecation-89463-SwitchableControllerActions.html)
in TYPO3 10.4 and will be removed in either TYPO3 11 but most likely 12, I decided to remove switchableControllerActions
in my TYPO3 Extbase extensions already with the upcoming versions that will be compatible with TYPO3 11.

In this blogpost I will show, how extension authors can add a smooth migration path to their existing extensions by
adding an update wizard which migrates all existing plugin settings, so users do not have to change plugin settings
manually.

As a starting point lets have a look at my TYPO3
extension [Plain FAQ](https://extensions.typo3.org/extension/plain_faq/), which is a very simple Extbase extension with
one plugin, that has 3 switchableControllerActions.

Source: [Flexform for Pi1](https://github.com/derhansen/plain_faq/blob/2.x/Configuration/FlexForms/Pi1.xml#L15)

*   Faq->list;Faq->detail
*   Faq->list
*   Faq->detail

For all 3 switchableControllerActions, I created 3 individual plugins _(Pilistdetail, Pilist, Pidetail)_ which handle
the action(s) of each switchable controller action from the list above.

For each new plugin, I added an individual FlexForm file which holds the available settings for the plugin. This can be
done by duplicating the old FlexForm (Pi1 in this case) and removing those settings, which are not available in the new
plugin. Also display conditions based switchableControllerActions must be removed.

Finally I created a new item group for the Plugins of the extension, so all plugins are grouped as shown on the
screenshot below.

![](/assets/images/2021-03-22/image1.png)

This is basically all work that needs to be done on code side in order split the old plugin to the new plugins.

### Migration of existing plugins

To be able to migrate all existing plugins and settings to the new plugins, I created a custom upgrade wizard that takes
care of all required tasks. Those tasks are as following:

* Determine, which tt\_content record need to be updated
* Analyse existing Plugin (field: _list\_type_) and switchableControllerActions in FlexForm (field: _pi\_flexform_)
* Remove non-existing settings and switchableControllerAction from FlexForm by comparing settings with new FlexForm
  structure of target plugin
* Update tt\_content record with new Plugin and FlexForm

As a result,
a **[SwitchableControllerActionsPluginUpdater](https://gist.github.com/derhansen/4524495ccfef9335c96d6d535bad7324)** has
been added to the extension. It takes care of all mentioned tasks and has a configuration array which contains required
settings (_source plugin, target plugin and switchableControllerActions_) for the migration.

{% highlight php %}
private const MIGRATION_SETTINGS = [
    [
        'sourceListType' => 'plainfaq_pi1',
        'switchableControllerActions' => 'Faq->list;Faq->detail',
        'targetListType' => 'plainfaq_pilistdetail'
    ],
    [
        'sourceListType' => 'plainfaq_pi1',
        'switchableControllerActions' => 'Faq->list',
        'targetListType' => 'plainfaq_pilist'
    ],
    [
        'sourceListType' => 'plainfaq_pi1',
        'switchableControllerActions' => 'Faq->detail',
        'targetListType' => 'plainfaq_pidetail'
    ],
];
{% endhighlight %}

So basically, one configuration entry has to be added for each switchable controller action setting of the old plugin.
The wizard determines the new FlexForm settings using configured TCA, removes all non-existing settings (which is
important, since TYPO3 will pass every setting available in pi\_flexform to Extbase controllers and Fluid templates) and
changes the "old" Plugin to the new one.

The update wizard can possibly also be used in other Extbase extensions, since the MIGRATION\_SETTINGS are the only
configuration options that need to be changed.

The required changes for the complete removal of switchableControllerActions is available in
this [commit](https://github.com/derhansen/plain_faq/commit/87acf8f61133383165d5a02881b227f4de43997d).