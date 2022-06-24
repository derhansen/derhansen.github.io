---
layout: post
title: How to manually create the default crop variant string for an imported image
  in TYPO3 CMS
date: '2021-12-29T12:30:00.001+01:00'
author: Torben Hansen
tags:
- import
- crop variant
- TYPO3
- sys_file_reference
modified_time: '2021-12-29T14:28:38.653+01:00'
blogger_id: tag:blogger.com,1999:blog-6517038209122183182.post-5015708534014158737
blogger_orig_url: http://www.derhansen.de/2021/12/typo3-image-import-crop-variant-string.html
permalink: /2021/12/typo3-image-import-crop-variant-string.html
---

When data in TYPO3 is created automatically (e.g. through a custom API or by an import script), it is very common, that also new files (especially images) are imported. TYPO3 has the well documented FAL (File Abstraction Layer), which provides an API for common tasks. 

One typical task is to **import an image** to FAL and next creating a file reference for the imported image to a record (e.g. event record of ext:sf\_event\_mgt). Such a task is easy to implement in a custom extension when you follow the example [documentation](https://docs.typo3.org/m/typo3/reference-coreapi/main/en-us/ApiOverview/Fal/UsingFal/ExamplesFileFolder.html) from the TYPO3 FAL API. This all works fine as described, as long as the referenced image does not use [Crop Variants](https://docs.typo3.org/m/typo3/reference-coreapi/10.4/en-us/ApiOverview/CropVariants/General/Index.html). For image fields, where crop variants are configured, **no crop variants will be created** for imported images and as a result, TYPO3 will always use the _aspect ratio of the imported image_ no matter which crop variant is configured for output.

#### TYPO3 internals

Available crop variants for image fields are defined in TCA. When an editor adds an image to a record in the TYPO3 backend, TYPO3 will automatically calculate the default crop variants. The result is saved to the table _sys\_file\_reference_ in the field _crop_ as a **JSON encoded string** like the shown example crop variant string below:

{% highlight json %}
{"heroimage":{"cropArea":{"x":0,"y":0.23,"width":1,"height":0.54},"selectedRatio":"16:9","focusArea":null},"teaserimage":{"cropArea":{"x":0,"y":0.14,"width":1,"height":0.72},"selectedRatio":"4:3","focusArea":null}}
{% endhighlight %}

This process if performed in [ImageManipulationElement](https://github.com/TYPO3/typo3/blob/v11.5.4/typo3/sysext/backend/Classes/Form/Element/ImageManipulationElement.php#L158). When an editor for example edits a record with an imported image and opens an image inline element, then the ImageManipulationElement is rendered and the default crop variant string is calculated for the imported image and saved, as soon as the editor saves the record.

#### Manually calculating the default crop variant string

In order to manually calculate the default crop variant string in an image import process, it is required to extract 2 code snippets from the _ImageManipulationElement_, since both are not public available:

1.  The default crop configuration - see [code](https://github.com/TYPO3/typo3/blob/v11.5.4/typo3/sysext/backend/Classes/Form/Element/ImageManipulationElement.php#L50)
2.  The function _populateConfiguration_ - see [code](https://github.com/TYPO3/typo3/blob/v11.5.4/typo3/sysext/backend/Classes/Form/Element/ImageManipulationElement.php#L262)

Both is extracted to a class named _CropVariantUtility_ and an additional public function is added which returns the crop variant string for a given file as shown below:

{% highlight php %}
/**
 * Returns a crop variant string to be used in sys_file_reference field "crop" for the given file and table/fieldname
 *
 * @param File $file
 * @param string $tableName
 * @param string $fieldname
 * @return string
 * @throws InvalidConfigurationException
 */
public static function getCropVariantString(File $file, string $tableName, string $fieldname): string
{
    $config = $GLOBALS['TCA'][$tableName]['columns'][$fieldname]['config']['overrideChildTca']['columns']['crop']['config'];
    $cropVariants = self::populateConfiguration($config);
    $cropVariantCollection = CropVariantCollection::create('', $cropVariants['cropVariants']);
    if (!empty($file->getProperty('width'))) {
        $cropVariantCollection = $cropVariantCollection->applyRatioRestrictionToSelectedCropArea($file);
    }

    return (string)$cropVariantCollection;
}
{% endhighlight %}

The whole class is available in this [gist](https://gist.github.com/derhansen/2ce92d64eccc802daa8212b5a0136b09).

Finally the new _CropVariantUtility_ can be used in a file import routine as shown in the example below:

{% highlight php %}
protected function addFileToEvent(File $file, int $eventUid): void
{
    $eventRecord = BackendUtility::getRecord(self::EVENT_TABLE, $eventUid);

    $fileReferenceUid = StringUtility::getUniqueId('NEW');

    $dataMap = [];
    $dataMap['sys_file_reference'][$fileReferenceUid] = [
        'table_local' => 'sys_file',
        'tablenames' => self::EVENT_TABLE,
        'uid_foreign' => $eventUid,
        'uid_local' => $file->getUid(),
        'fieldname' => 'image',
        'pid' => $eventRecord['pid'],
        'show_in_views' => 0,
        'crop' => CropVariantUtility::getCropVariantString($file, self::EVENT_TABLE, 'image'),
    ];

    $dataMap[self::EVENT_TABLE][$eventUid] = [
        'image' => $fileReferenceUid
    ];

    $this->dataHandler->start($dataMap, []);
    $this->dataHandler->process_datamap();
}
{% endhighlight %}

The example code will create a new file reference for the given file object for the table  _self::EVENT\_TABLE_ (
tx\_sfeventmgt\_domain\_model\_event in this case) and the given event UID. The usage of the new _CropVariantUtility_
ensures, that the new file relation has a **default crop variant string** and configured crop variants can directly be
used for imported images.