---
layout: post
title: TYPO3 6.2 - Problems with GIFBUILDER and filenames containing umlauts
date: '2015-01-17T07:39:00.001+01:00'
author: Torben Hansen
tags:
- GifBuilder
- UTF8 filesystem
- umlauts
- TYPO3 6.2
modified_time: '2015-01-29T16:39:47.298+01:00'
blogger_id: tag:blogger.com,1999:blog-6517038209122183182.post-2912592419267973251
blogger_orig_url: http://www.derhansen.de/2015/01/typo3-62-problems-with-gifbuilder-and.html
permalink: /2015/01/typo3-62-problems-with-gifbuilder-and.html
---

After migrating a TYPO3 4.5 website to TYPO3 6.2 I had some problems in a third party extension, which resulted in the
following exception being thrown.

{% highlight text %}
#1: PHP Warning: imagecreatefromgif(typo3temp/_processed_/csm_t%C3%A4st.gif): failed to open stream: No such file or directory in /path-to-typo3/typo3_src-6.2.9/typo3/sysext/core/Classes/Imaging/GraphicalFunctions.php line 2873 
{% endhighlight %}

I did some debugging and found out, that the exception was thrown, because the extension used the result of an **
IMG\_RESOURCE** object (which in this case is the path to an resized image) for further TypoScript image processing. The
special thing about it was, that the exception was only thrown for images where the filename contained **umlauts**.

Generally, I never use umlauts in filenames and if TYPO3 is not configured to use an utf-8 to store filenames, **TYPO3
automatically converts** umlauts (e.g. "ä" to "ae"). The setup in this TYPO3 installation was different and in the
install tool the following settings were present.

{% highlight php %}
[SYS][UTF8filesystem] = 1
[SYS][systemLocale] = de_DE.utf8
{% endhighlight %}

The code in the third party extension was fine for TYPO3 4.5, since resized and temporary images did not include the
original filename. For TYPO3 6.2, this is different in some situations and resized/temporary images may get **"csm\_"**
as prefix followed by the original filename. This can result in some strange behaviour with images not being rendered as
intended. I will demonstrate this below.

My test setup uses TYPO3 6.2.9, PHP 5.5 and I have configured \[SYS\]\[UTF8filesystem\] = 1 and \[SYS\]\[systemLocale\]
= de\_DE.utf8.

First I uploaded a file named "test.jpg" and assigned it through TypoScript to the frontend output as shown below.

{% highlight php %}
lib.test = IMAGE
lib.test.file = fileadmin/gfx/test.jpg
{% endhighlight %}

There is nothing special about this and as you may expect, the image was successfully rendered in the frontend.

Next I uploaded a file names "täst.jpg" and did the same as shown above.

{% highlight php %}
lib.test = IMAGE
lib.test.file = fileadmin/gfx/täst.jpg
{% endhighlight %}

Also here, the image was rendered successfully in the frontend. A look in the sourcecode did show, that the umlaut image
filename has been **urlencoded**.

{% highlight html %}
<img alt="" border="0" height="250" src="fileadmin/gfx/t%C3%A4st.jpg" width="250" />
{% endhighlight %}

This all works fine for simple TypoScript IMAGE objects. Now lets get our good, old friend GIFBUILDER into the team :-)

The following TypoScript is kept simple for demostration purposes.

{% highlight php %}
lib.test = IMAGE
lib.test {
  file = GIFBUILDER
  file {
    XY = 100,100
    10 = IMAGE
    10.file = fileadmin/gfx/test.jpg
    20 = TEXT
    20.text = Test
    20.fontColor = #000000
    20.fontFile = fileadmin/vera.ttf
    20.offset = 10,20
  }
}
{% endhighlight %}

The output of the TypoScript above is the original image "test.jpg", where the text "Test" is rendered on top of the
image.

As the last step, I replace the image **"test.jpg"** with the image **"täst.jpg"**.

{% highlight php %}
lib.test = IMAGE
lib.test {
  file = GIFBUILDER
  file {
    XY = 100,100
    10 = IMAGE
    10.file = fileadmin/gfx/täst.jpg
    20 = TEXT
    20.text = Test
    20.fontColor = #000000
    20.fontFile = fileadmin/vera.ttf
    20.offset = 10,20
  }
}
{% endhighlight %}

What would you expect to be rendered? I guess, the same result as for the previous example. Actually, the final image is
just a GIF file, which has a white background and shows the text "Test". The **image** with the umlaut is
**not processed**.

So why does TYPO3 not render the image, when umlauts are used in the filename? The main reason for this behaviour is
located in **GIFBUILDER,** which uses image path that is rawurlencoded.

**1.** When GIFBUILDER starts to render the final image, it iterates through all TypoScript objects inside it. For IMAGE
objects, the GIFBUILDER uses it's own function **getResource($file, $fileArray)** to return a GIFBUILDER image.

{% highlight php %}
case 'IMAGE':
 $fileInfo = $this->getResource($conf['file'], $conf['file.']);
 if ($fileInfo) {
  $this->combinedFileNames[] = preg_replace('/\\.[[:alnum:]]+$/', '', basename($fileInfo[3]));
  $this->setup[$theKey . '.']['file'] = $fileInfo[3];
{% endhighlight %}

**2\.** The function getResource uses the the function **getImgResource** from the **contentObjectRenderer** to create
get the (resized/scaled) image resource for the file.

{% highlight php %}
public function getResource($file, $fileArray) {
 if (!GeneralUtility::inList($this->imageFileExt, $fileArray['ext'])) {
  $fileArray['ext'] = $this->gifExtension;
 }
 /** @var \TYPO3\CMS\Frontend\ContentObject\ContentObjectRenderer $cObj */
 $cObj = GeneralUtility::makeInstance('TYPO3\\CMS\\Frontend\\ContentObject\\ContentObjectRenderer');
 $cObj->start($this->data);
 return $cObj->getImgResource($file, $fileArray);
}
{% endhighlight %}

**3\.** The function getImgResource resizes/scales the given image, saves the processed image as **
/fileadmin/\_processed\_/csm\_täst\_9f3d74f15e.gif** and returns an array including a lot of information (see below)
about the processed file.

{% highlight php %}
if ($processedFileObject->isProcessed() && !isset($GLOBALS['TSFE']->tmpl->fileCache[$hash])) {
 $GLOBALS['TSFE']->tmpl->fileCache[$hash] = array(
  0 => $processedFileObject->getProperty('width'),
  1 => $processedFileObject->getProperty('height'),
  2 => $processedFileObject->getExtension(),
  3 => $processedFileObject->getPublicUrl(),
  'origFile' => $fileObject->getPublicUrl(),
  'origFile_mtime' => $fileObject->getModificationTime(),
  // This is needed by \TYPO3\CMS\Frontend\Imaging\GifBuilder,
  // in order for the setup-array to create a unique filename hash.
  'originalFile' => $fileObject,
  'processedFile' => $processedFileObject,
  'fileCacheHash' => $hash
 );
}
$imageResource = $GLOBALS['TSFE']->tmpl->fileCache[$hash];
{% endhighlight %}

4\. The interesting part here is **$processedFileObject->getPublicUrl()**, which returns the public URL for the
processed FAL image.

{% highlight php %}
public function getPublicUrl($identifier) {
 $publicUrl = NULL;
 if ($this->baseUri !== NULL) {
  $uriParts = explode('/', ltrim($identifier, '/'));
  $uriParts = array_map('rawurlencode', $uriParts);
  $identifier = implode('/', $uriParts);
  $publicUrl = $this->baseUri . $identifier;
 }
 return $publicUrl;
}
{% endhighlight %}

Array\_map is used on all $uriParts with **rawurlencode()** to ensure the path to the image is encoded correctly. For my
testfile "täst.jpg", the function returns "fileadmin/\_processed\_/csm\_t%C3%A4st\_9f3d74f15e.gif".

Now going back to **step 1** where the GIFBUILDER iterates through all TypoScript objects you will see, that
$fileInfo\[3\] is used as a path to the image and this is what results in the GIFBUILDER object not being rendered
correctly. As the **filename** is **urlencoded**, it can't be used by GIFBUILDER for futher processing which in this
case is rendering the text "Test" on it.

**Conclusion**

The problem only appears under rare curcumstances (using \[SYS\]\[UTF8filesystem\] = 1), and I believe that not many
people are affected of it, since TYPO3 by default has disabled this feature. It seems, this is a bug in TYPO3, which
actually was [reported](https://forge.typo3.org/issues/64224) by Peter Niederlag **nearly at the same time** I struggled
with the problem. I'll join the discussion and am pretty sure that a bugfix will be available soon.

**Update 29.01.2015:** The bugfix for the problem described above has been merged today for TYPO3 6.2 and current
master.