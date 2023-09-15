---
layout: post
title: 'The significance of the PHP exif module in TYPO3 and why it should always be enabled'
date: '2023-09-14T15:00:00.000+02:00'
author: Torben Hansen
tags:
- typo3
- image processing 
- distorted image
- exif
modified_time: '2023-09-15T10:10:00.000+02:00'
permalink: /2023/09/the-significance-of-the-php-exif-module-in-typo3-why-you-should-always-use-it.html
---

**TL;DR:** If using original portrait images created with modern camera systems, you should ensure that 
the PHP exif module is active, so that portrait images do not get distorted.

People who know me most likely also know, that I like to work on challenging and complex TYPO3 tasks. Yesterday, 
I faced an unusual situation in TYPO3, which took me some time to resolve and where the solution most likely is not
very well known.

In a TYPO3 project which recently has been migrated to TYPO3 11.5 it occured, that some portrait images
did not get resized correctly. Instead, those images where distorted, as soon as TYPO3 image processing
functionality was involved. 

The original image was created with a Motorola smartphone in portrait mode. When opened on a Mac or in a 
webbrowser, the image just looks as it should. 

When the image was uploaded to TYPO3 and used in a TYPO3 text/media content element, where the 
image width have been change to e.g. 600 pixel width, the image was distorted as shown to the right on the 
screenshot below:

![Image comparision - distorted portrait image in TYPO3](/assets/images/2023-09-20/typo3-image-distorted.png)

The problem however only appeared on some webservers and all my public and local testing-environments
dit not create the shown result and always rendered the image in portait mode without any distortion.

After some time of debugging and excluding involved tools like ImageMagick or GraphicsMagic and their
settings in TYPO3, I searched on public forums and TYPO3 slack for similar problems. I only found one 
discusson of a similar situation, which was related to the **PHP module exif**. So I checked, if the PHP 
version on the affected webserver had the PHP exif module active and this was actually **not** the case. 

After enabling the PHP module exif and after re-uploading the image, everything worked and looked as expected.

### In detail analysis 

After resolving the main problem, I was interested in, why the image was only processed correctly when the
PHP exif module was installed. 

#### The image

Since not all portrait images where distorted on the affected website, I first analyzed the original image by using 
the [exiftool](https://exiftool.org/index.html){:target="_blank"} to get the exif metadata. 
Below are the most important attributes:

{% highlight text %}
File Size                       : 15 MB
Orientation                     : Rotate 90 CW
Image Size                      : 8192x4608
{% endhighlight %}

Modern camera systems use a combination of hardware and software features to determine the orientation of a photo.
So when a photo is taken with e.g. an Motorola Edge 30 Ultra, it records information about the orientation in the 
image's EXIF data, including whether it was taken in portrait or landscape mode. So the shown size of the image 
is actually correct, but the `Orientation` property determines, that it must be **rotated 90° clockwise**.

#### Image upload in TYPO3

When an image in TYPO3 is uploaded, TYPO3 detects the size of an image and saves the width and height if the file 
in the table `sys_file_metadata`. First, TYPO3 tries to determine the image size using the PHP function `getimagesize()`.
For the original image referred in this acticle, the width is 8192 pixel and the hight is 4608 pixel.

If the PHP module exif is present, TYPO3 will additionally determine the image size using `exif_read_data()` and will 
then check the `Orientation` exif property. If the property defines, that the image should be rotated, TYPO3 will switch
the width and hight of the image and will save, that the image is 4608 pixel width and 8192 pixel height.

And this is also the explanation, why the image is distorted, when PHP exif module is missing. TYPO3 does not know about
the image orientation and will save the image size as the `getimagesize()` function returns. The screenshot below show,
that the orientation is not respected, if the PHP exif extension is missing:

![Wrong image sizes](/assets/images/2023-09-20/typo3-wrong-image-size.png)

Those determined image sizes are later used, when TYPO3 processes the file e.g. in order to render the image in a
smaller size, which then results in the distorted image effect as shown initially. 

### Conclusion

If you use original images created by digital cameras in TYPO3, you should always ensure, that the PHP module exif is 
installed and active, so the exif `Orientation` property is respected. 

For the rescue, most webhosters and also linux distributions (e.g. Ubuntu 22.04) already have PHP exif module 
enabled by default. 