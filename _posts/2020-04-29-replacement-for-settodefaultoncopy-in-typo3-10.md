---
layout: post
title: How to add a replacement for the removed TCA Option "setToDefaultOnCopy" in
  TYPO3 10.4
date: '2020-04-29T20:14:00.000+02:00'
author: Torben Hansen
tags:
- TYPO3
- TCA
- setToDefaultOnCopy
modified_time: '2020-04-29T20:15:42.332+02:00'
blogger_id: tag:blogger.com,1999:blog-6517038209122183182.post-6277615038991766885
blogger_orig_url: http://www.derhansen.de/2020/04/replacement-for-settodefaultoncopy-in-typo3-10.html
permalink: /2020/04/replacement-for-settodefaultoncopy-in-typo3-10.html
---

The TYPO3 TCA Option "setToDefaultOnCopy" has been [removed](https://forge.typo3.org/issues/87989) in TYPO3 10 in order
to reduce the amount of checks in DataHandler and the amount of available options in TCA. The documentation says,
that _"This option was only there for resetting some \`sys\_action\` values to default, which_ _can easily be achieved
by a hook if needed. If an extension author uses this setting,_  
_this should be achieved with proper DataHandler hooks."_  
I use this option in one of my extensions. Basically, I have one "main" record, that has one Inline field with various "
subrecords". Those "subrecords" are user generated and should not be copied, when the main record is copied, so I had to
find out which DataHandler hooks should be used to get the removed functionality back for the TYPO3 10 compatible
version of my extension.

After some hours with several breakpoints in the TYPO3 DataHandler I came to the conclusion, that this may not be as "
easy" as described, since there is no Hook, where you can unset certain field values during the copy (or localize)
process. And if there was, then another problem would have shown up, since relation fields are processed different (
basically the relation is resolved using TCA) on copy or translation commands in DataHandler.

Knowing the last about TCA however makes it possible to hook into the process. At a very early stage in DataHandler, I
use processCmdmap\_preProcess to set the TCA type for affected fields to "none" as shown below:

{% highlight php %}
public function processCmdmap_preProcess($command, $table, $id, $value, $pObj, $pasteUpdate)
{
    if (in_array($command, ['copy', 'localize']) && $table === 'tx_extension_table') {
        $GLOBALS['TCA']['tx_extension_table']['columns']['fieldname1']['config']['type'] = 'none';
        $GLOBALS['TCA']['tx_extension_table']['columns']['fieldname2']['config']['type'] = 'none';
    }
}
{% endhighlight %}

With this configuration in TCA, the affected fields are completely ignored by the copy/localize command in DataHandler.
It is now just important to change the field types back after the command is finished in processCmdmap\_postProcess hook
as shown below:

{% highlight php %}
public function processCmdmap_postProcess($command, $table, $id, $value, $pObj, $pasteUpdate, $pasteDatamap)
{
    if (in_array($command, ['copy', 'localize']) && $table === 'tx_extension_table') {
        $GLOBALS['TCA']['tx_extension_table']['columns']['fieldname1']['config']['type'] = 'text';
        $GLOBALS['TCA']['tx_extension_table']['columns']['fieldname2']['config']['type'] = 'inline';
    }
}
{% endhighlight %}

Hard to say, if this is a good approach to get the functionality back. It feels not really right to change existing TCA
at runtime as shown, but at least, I could not find any downsides in the solution and it works fine for me.