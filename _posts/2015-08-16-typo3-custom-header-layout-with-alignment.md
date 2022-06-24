---
layout: post
title: TYPO3 6.2 - How to create a custom header layout and keep the alignment field
  working
date: '2015-08-16T13:35:00.001+02:00'
author: Torben Hansen
tags:
- alignment
- TYPO3
- custom header
modified_time: '2015-08-16T13:39:12.518+02:00'
thumbnail: http://3.bp.blogspot.com/-2cVdb0J8rq4/Vc8nbNdt_hI/AAAAAAAAPoc/jDVOcgjnwiM/s72-c/Bildschirmfoto%2B2015-08-15%2Bum%2B13.49.31.png
blogger_id: tag:blogger.com,1999:blog-6517038209122183182.post-3697271603460233544
blogger_orig_url: http://www.derhansen.de/2015/08/typo3-custom-header-layout-with-alignment.html
permalink: /2015/08/typo3-custom-header-layout-with-alignment.html
---

**tl;dr:** When you add custom headers layouts to TYPO3, always keep in mind to **retain original functionality** of
the **alignment select field** as shown at the end of this blogpost (_solution 1_ or _solution 2_), so editors don't get
confused.

If you use **CSS Styled Content** in TYPO3, there are many tutorials on the internet showing how to configure **custom
headers**. So why do I write another article about this you may ask. Well, in a project I "just" needed to create a **
custom header layout**, where the user could use the TYPO3 **"alignment"-field** to select the alignment of the custom
header. The resulting HTML should include the class "custom\_class" like shown below:

{% highlight html %}
<h1 class="custom_class">TYPO3 Content Management System</h1>
{% endhighlight %}

After some time of unsuccessful research, I finally ended up with the solutions shown below (at the end of the 
blogpost).

### The standard solution (not recommended)

Besides many **non-working** or **outdated solutions** (e.g. _{register:headerStyle}_ has been deprecated with TYPO3 4.7
I guess) you may find solutions which instruct you to do as following:

1\. Define a custom header style in Page TSConfig.

{% highlight php %}
TCEFORM.tt_content.header_layout {
    addItems.10 = Header with custom layout
}
{% endhighlight %}

2\. Add TypoScript to inherit the header style from lib.stdheader.10.1 and finally overwrite the dataWrap attribute.

{% highlight php %}
lib.stdheader.10.10 < lib.stdheader.10.1
lib.stdheader.10.10 {
  dataWrap = <h1 class="custom_class">|</h1>
}
{% endhighlight %}

What is wrong with this solution? Well, did you ever try to set the **header alignment** for a custom header, which is
configured as shown above?

![](/assets/images/2015-08-16/image1.png)

**It does'nt work**! The **editor may get confused** and claim, that TYPO3 is not working correct, because the alignment
selection in the backend doesn't affect the frontend.

### Why is the alignment not working?

In order to remain the alignment that is set by CSS Styled Content, we need to understand how CSS Styled Content
technically sets it. When the editor sets the alignment of a header to **"Center"**, the resulting **HTML** **header
tag** will include the CSS class **"csc-header-alignment-center"**, which controls the header alignment in the frontend.

What happens "under the hood" is as following: A look into the **TypoScript of CSS Styled Content** from TYPO3 6.2.14
shows,
that [header alignments](https://github.com/TYPO3/TYPO3.CMS/blob/6.2.14/typo3/sysext/css_styled_content/static/setup.txt#L182)
are loaded into a register called _headerClass_, which is used in
the [dataWrap](https://github.com/TYPO3/TYPO3.CMS/blob/6.2.14/typo3/sysext/css_styled_content/static/setup.txt#L237)
attribute to set all classes for the resulting header. And this is the reason, why the example above does not work as
expected. **Overwriting** the **dataWrap** attribute **removes the alignment** set by CSS Styled Content.

### Solution 1 - extending the headerClass register

As we now know, that we need to include the register _headerClass_ in the dataWrap attribute to keep the alignment, we
just extend the value of the headerClass register.

1\. Define a custom header style in Page TSConfig.

{% highlight php %}
TCEFORM.tt_content.header_layout {
    addItems.10 = Header with custom layout
}
{% endhighlight %}

2\. Load the _original register_ into a **new register** (headerClassFor10) and add a new TEXT content object to the
existing COA (content object array)

{% highlight php %}
lib.stdheader.3.headerClassFor10 < lib.stdheader.3.headerClass
lib.stdheader.3.headerClassFor10.cObject {
    5 = TEXT
    5.noTrimWrap = |custom_class ||
}
lib.stdheader.10.10 < lib.stdheader.10.1
lib.stdheader.10.10.dataWrap = <h1{register:headerClassFor10}>|</h1>
{% endhighlight %}

The resulting header tag now includes the CSS classes set by CSS Styled Content and the new class "custom\_class".

{% highlight php %}
<h1 class="custom_class csc-header-alignment-center">TYPO3 Content Management System</h1>
{% endhighlight %}

If you have several custom header layouts, make sure, that you define a new register for each custom header layout (
e.g. **headerClassFor10** and **headerClassFor11** in case you have defined header **10** and **11)**.

### Solution 2 - extending the dataWrap with a span-tag

If you don't like solution 1, you can use the solution shown below, which just adds a **HTML span tag inside** the **
header tag** (which is valid for both HTML4 and HTML5). This time, I just show you how the TypoScript for the custom
header should look like (you've already seen the Page TSConfig 2 times).

{% highlight php %}
lib.stdheader.10.10 < lib.stdheader.10.1
lib.stdheader.10.10 {
  dataWrap = <h1{register:headerClass}><span class="custom_class">|</span></h1>
}
{% endhighlight %}

The resulting HTML output for this header is as following:

{% highlight html %}
<h1 class="csc-header-alignment-center"><span class="custom_class">TYPO3 Content Management System</span></h1>
{% endhighlight %}