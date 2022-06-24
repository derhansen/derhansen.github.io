---
layout: post
title: TYPO3 8.7 LTS - How to enable the image cropping tool in your own extension
date: '2017-06-28T13:49:00.000+02:00'
author: Torben Hansen
tags:
- Image Cropping
- typo3 8.7
- Extbase
modified_time: '2017-06-28T13:52:04.458+02:00'
blogger_id: tag:blogger.com,1999:blog-6517038209122183182.post-6076001017105213347
blogger_orig_url: http://www.derhansen.de/2017/06/typo3-87-enable-image-cropping-tool-own-extension.html
permalink: /2017/06/typo3-87-enable-image-cropping-tool-own-extension.html
---

People using TYPO3 7.6 or 8.7 for sure know the cool Image Cropping tool that was introduced with TYPO3 7. As an
extension developer, you can easily switch on the Image Cropping tool for your own extension by enabling adding the _
imageoverlayPalette_ to the _foreign\_types_ config array for the _FILETYPE\_IMAGE_ in the TCA like shown
in [this blogpost](https://typo3worx.eu/2016/02/image-cropping-typo3-backend/) by Marcus Schwemer. In TYPO3 8.7 the 
**Image Cropping tool does not show** as expected, due to structural TCA changes in the TYPO3 8.

For TYPO3 8.7, the following (minimal) TCA configuration will enable the Image Cropping tool for the field "image" in an
own extension:

{% highlight php %}
'image' => [
    'exclude' => 1,
    'label' => 'LLL:EXT:custom_extension/Resources/Private/Language/locallang_db.xlf:tx_customextension_domain_model_tablename.custom_field',
    'config' => \TYPO3\CMS\Core\Utility\ExtensionManagementUtility::getFileFieldTCAConfig('image', [
        'foreign_match_fields' => [
            'fieldname' => 'image',
            'tablenames' => 'tx_customextension_domain_model_tablename',
            'table_local' => 'sys_file',
        ],
        'overrideChildTca' => [
            'types' => [
                \TYPO3\CMS\Core\Resource\File::FILETYPE_IMAGE => [
                    'showitem' => '
                            --palette--;LLL:EXT:lang/Resources/Private/Language/locallang_tca.xlf:sys_file_reference.imageoverlayPalette;imageoverlayPalette,
                            --palette--;;filePalette'
                ],
            ],
        ],
        'minitems' => 0,
        'maxitems' => 999,
    ], $GLOBALS['TYPO3_CONF_VARS']['GFX']['imagefile_ext']),
],
{% endhighlight %}

Note, that the configuration array only shows the configuration for a single field and that the array must be located in
the **"columns"-array** of your domain model TCA. For TYPO3 8.7, the **overrideChildTca** config is the important part,
which enables the Image Cropping tool.

If your extension needs to support TYPO3 7.6 and 8.7, your TCA should include both the **overrideChildTca** config as
described in this blogpost and the **foreign\_types** config as described in Marcus's blogpost.