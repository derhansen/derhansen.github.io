---
layout: post
title: How Extbase determines the storagePid setting for a plugin
date: '2016-02-08T08:00:00.000+01:00'
author: Torben Hansen
tags:
- TYPO3 7.6
- Extbase
- storagePid
- TYPO3 6.2
modified_time: '2016-04-17T19:06:53.787+02:00'
blogger_id: tag:blogger.com,1999:blog-6517038209122183182.post-849144282647254052
blogger_orig_url: http://www.derhansen.de/2016/02/how-extbase-determines-storagepid.html
permalink: /2016/02/how-extbase-determines-storagepid.html
---

When you search for the terms "extbase storagePid" in your favorite search engine, you will find several blogs,
forum-posts, mailing list entries and code examples on how to set the storagePid for the plugin of an Extbase extension.
Many examples refer to TypoScript persistence settings for the extension itself (**
plugin.tx\_myext.persistence.storagePid****)**, which is good, as long as you do not _directly_ set persistence settings
for a plugin like shown in the example.

### Example

Let us assume, that you have a TYPO3 Extbase extension, that has configured a storagePid for a plugin in TypoScript as
shown below.

{% highlight php %}
plugin.tx_myext_myplugin {
 persistence {
  storagePid = {$plugin.tx_myext_myplugin.persistence.storagePid}
 }
}
{% endhighlight %}

Now you want the editor to be able to set the storagePid from the plugin settings. Since a TYPO3 plugin by default has a
configuration setting called **Record Storage Page,** you use this configuration setting and select a page.

Surprisingly, the Plugin will still use the setting configured in TypoScript. But why?

### Extbase storagePid determination

Extbase uses the configurationManager to determine the storagePid. When you create an Extbase extension which should be
configurable by users, it is important to understand **how and in which order** Extbase **determines/overrides** the
storagePid for a plugin.

#### Step 1 - Default storagePid

Extbase uses the default storagePid which is 0

#### Step 2 - Extbase persistence settings

If Extbase has configured a storagePid in **config.tx\_extbase.persistence.storagePid**, then Extbase will use this
storagePid. _(note, in the code, this happens before before step 1)_

#### Step 3 - Extension/Plugin configuration from TypoScript

Extbase fetches the configuration for the **plugin** from TypoScript persistence settings in **
`plugin.tx_myext.persistence.storagePid` _and_ _merges it_ _with_ `plugin.tx_myext_myplugin.persistence.storagePid`
\- The plugin settings override the extension settings. If the resulting storagePid is set (or empty), Extbase will use
the configured value as storagePid

#### Step 4 - Override storagePid from starting point setting

If the plugin uses the starting point (Record Storage Page) and one/multiple pages are selected as starting point,
Extbase will now use this as storagePid.

#### Step 5 - Override storagePid from Plugin settings

Now Extbase fetches the configuration for the **plugin** from TypoScript from 
`plugin.tx_myext_myplugin.persistence.storagePid` and if set (or empty), Extbase now uses this as storagePid.

#### Step 6 - Override storagePid from Flexform

Finally Extbase checks, if the plugin has configured a flexform and if so, it merges all **settings**, **persistence**
and **view** elements from the flexform with the current Extbase plugin configuration. The merge is done recursive and
settings from TypoScript will be overridden by the flexform settings. So if you have configured **
persistence.storagePid** in your flexform, this will be used as the storagePid.

Make sure you read the **important note** below, since there is something to keep in mind when working with empty
flexform values.

#### Step 7 - Expand storagePid if recursive setting is set

If `persistence.recursive` is set in `plugin.tx_myext_myplugin` or in your **flexform,** Extbase will expand all
configured storagePids recursive by the configured depth. If set, the final storagePid will contain a list of page uids
for all subfolders of the original storagePid.

It is important to know, that each determination step will **override** the storagePid setting from the previous step.
So for our example from above, the starting point setting gets overridden by the Plugin TypoScript setting.

Also keep in mind, that as soon as you use `plugin.tx_myext_myplugin.persistence.storagePid`, the persistence
settings for the extension `plugin.tx_myext.persistence.storagePid` and also the **record storage page settings**
will get **overridden.**

### StoragePid when creating new records

When you have set the storagePid (which in fact may contain several page uids), Extbase will always use the **first
uid** when creating a new record. So if you have configured  `plugin.tx_myext_myplugin.persistence.storagePid =
10,11,12` then new records created from the plugin will be saved to the page with the uid 10.

In order to override this behaviour, you can either add a property to your domain object which contains the page uid -
e.g. getPid() - or you can override the storagePid for new records by TypoScript with the newRecordStoragePid setting as
shown below:

{% highlight php %}
plugin.tx_myext {
 persistence {
  storagePid = {$plugin.tx_myext.persistence.storagePid}
  classes {
   Vendor\Myext\Domain\Model\Mymodel {
    newRecordStoragePid = 16
   }
  }
 }
}
{% endhighlight %}

### Setting the storagePid from flexform

To set the storagePid from flexform, you can use the code snippet below. It is an extract from a flexform XML structure
which contains a field for the storagePid and another field for the recursive setting.

{% highlight xml %}
<persistence.storagePid>
    <TCEforms>
        <exclude>1</exclude>
        <label>Storage PID</label>
        <config>
            <type>group</type>
            <internal_type>db</internal_type>
            <allowed>pages</allowed>
            <size>3</size>
            <maxitems>99</maxitems>
            <minitems>0</minitems>
            <show_thumbs>1</show_thumbs>
            <wizards>
                <suggest>
                    <type>suggest</type>
                </suggest>
            </wizards>
        </config>
    </TCEforms>
</persistence.storagePid>

<persistence.recursive>
    <TCEforms>
        <label>LLL:EXT:lang/locallang_general.xlf:LGL.recursive</label>
        <config>
            <type>select</type>
            <items type="array">
                <numIndex index="1" type="array">
                    <numIndex index="0">LLL:EXT:news/Resources/Private/Language/locallang_be.xlf:flexforms_general.recursive.I.inherit</numIndex>
                    <numIndex index="1"></numIndex>
                </numIndex>
                <numIndex index="2" type="array">
                    <numIndex index="0">LLL:EXT:frontend/Resources/Private/Language/locallang_ttc.xlf:recursive.I.0</numIndex>
                    <numIndex index="1">0</numIndex>
                </numIndex>
                <numIndex index="3" type="array">
                    <numIndex index="0">LLL:EXT:frontend/Resources/Private/Language/locallang_ttc.xlf:recursive.I.1</numIndex>
                    <numIndex index="1">1</numIndex>
                </numIndex>
                <numIndex index="4" type="array">
                    <numIndex index="0">LLL:EXT:frontend/Resources/Private/Language/locallang_ttc.xlf:recursive.I.2</numIndex>
                    <numIndex index="1">2</numIndex>
                </numIndex>
                <numIndex index="5" type="array">
                    <numIndex index="0">LLL:EXT:frontend/Resources/Private/Language/locallang_ttc.xlf:recursive.I.3</numIndex>
                    <numIndex index="1">3</numIndex>
                </numIndex>
                <numIndex index="6" type="array">
                    <numIndex index="0">LLL:EXT:frontend/Resources/Private/Language/locallang_ttc.xlf:recursive.I.4</numIndex>
                    <numIndex index="1">4</numIndex>
                </numIndex>
                <numIndex index="7" type="array">
                    <numIndex index="0">LLL:EXT:frontend/Resources/Private/Language/locallang_ttc.xlf:recursive.I.5</numIndex>
                    <numIndex index="1">250</numIndex>
                </numIndex>
            </items>
        </config>
    </TCEforms>
</persistence.recursive>
{% endhighlight %}

It is important, that the **field naming** of the fields is equal to the TypoScript naming. Since Extbase will merge all
flexform fields to the plugin configuration, **persistence.storagePid** and **persistence.recursive** from the flexform
will override the TypoScript settings.

**Important note:**  
If you have set a storagePid by flexform and _delete it afterwards_, then the final storagePid will be **empty**. The
reason for this is the field _pi\_flexform_ in _tt\_content_ table for the plugin, which will contain an empty value for
persistence.storagePid

{% highlight xml %}
<field index="persistence.storagePid">
  <value index="vDEF"></value>
</field>
{% endhighlight %}

This empty value actually overrides all other storagePid settings. You can avoid this behavior by using the **
processDatamap\_postProcessFieldArray hook** to unset empty field values in flexform if needed

### Conclusion

As a result of this blogpost, I would recommend to use `plugin.tx_myext.persistence.storagePid` and the 
**record storage page** option to set the storagePid of a plugin, since it works out of the box and does not 
have any things to keep mind.