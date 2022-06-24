---
layout: post
title: How to extend existing FlexForm select options of a TYPO3 plugin using Page
  TSconfig
date: '2020-11-16T16:16:00.005+01:00'
author: Torben Hansen
tags:
- flexform
- extend
- TYPO3
modified_time: '2020-11-16T16:23:45.847+01:00'
thumbnail: https://1.bp.blogspot.com/-2G8WD7LHQX8/X7KNv_pefBI/AAAAAAAAp9k/-enwp3LPHH8YkyKYwf29y_fiOrn9w-L2ACLcBGAsYHQ/s72-c/sf_event_mgt_sorting1.png
blogger_id: tag:blogger.com,1999:blog-6517038209122183182.post-8135798734812445400
blogger_orig_url: http://www.derhansen.de/2020/11/typo3-extend-existing-flexform-select-options.html
permalink: /2020/11/typo3-extend-existing-flexform-select-options.html
---

Sometimes existing options of a TYPO3 plugin may not fully suite the project requirements. As an example, I refer to my
TYPO3 extension "Event Management and
Registration" ([sf\_event\_mgt](https://extensions.typo3.org/extension/sf_event_mgt/)). The extension allows to select
the ordering of records by a specific field in the FlexForm plugin options as shown on the screenshot below.

![](/assets/images/2020-11-16/image1.png)

The 3 options shown are configured in the Flexform options for the field `settings.orderField`.

In a project it was required to order by a custom field which was not part of the main extension. So I added the custom
field named "slot" to the extension using
an [extending extension](https://github.com/derhansen/sf_event_mgt_extend_demo) for sf\_event\_mgt.

In order to allow the new field as sorting field, the field "slot" needs to be added to the allowed ordering fields
using TypoScript (note, this step is only specific to the extension sf\_event\_mgt).

{% highlight php %}
plugin.tx_sfeventmgt {
  settings {
    orderFieldAllowed = uid,title,startdate,enddate,slot
  }
}
{% endhighlight %}

Finally the FlexForm of the plugin needs to be extended, so the new field appears in the "Sort by" select field. In
order to do so, the following Page TSconfig has been added:

{% highlight php %}
TCEFORM.tt_content.pi_flexform.sfeventmgt_pievent.sDEF.settings\.orderField {
  addItems.slot = slot
  altLabels.slot = Slot
}
{% endhighlight %}

You might notice the **backslash** before the dot in `settings.orderField`. This is required to escape the dot of the
fieldname "settings.orderField", since Page TSconfig also uses dots to separate between configuration options/levels.

After adding the Page TSconfig, the plugin now shows the new field.

![](/assets/images/2020-11-16/image2.png)

Pretty cool and not a single line of PHP code required :-)

Reference: [TYPO3 TCEFORM](https://docs.typo3.org/m/typo3/reference-tsconfig/master/en-us/PageTsconfig/TceForm.html)