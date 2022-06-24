---
layout: post
title: TYPO3 Extbase - Manual validation of a domain model
date: '2017-06-26T11:01:00.000+02:00'
author: Torben Hansen
tags:
- property mapper
- Extbase
- validation
- TYPO3
- domain model
modified_time: '2017-06-26T11:01:10.501+02:00'
blogger_id: tag:blogger.com,1999:blog-6517038209122183182.post-5256957343753052897
blogger_orig_url: http://www.derhansen.de/2017/06/typo3-extbase-manual-domain-model-validation.html
permalink: /2017/06/typo3-extbase-manual-domain-model-validation.html
---

When you create an Extbase extension and allow a website user to submit form data that will be saved to the TYPO3
database, you usually work with validators in your domain model to ensure, that the given data will match certain
criteria (e.g. properties have the right data type or expected data format). Extbase offers an easy way to add
validation rules to domain model properties, by just adding the **@validate** annotation followed by one or multiple
validators as shown in the following example:

{% highlight php %}
/**
 * Description
 *
 * @var string
 * @validate NotEmpty, StringLength(minimum=10, maximum=50)
 */
protected $description = '';
{% endhighlight %}

Extbase will take care of the domain model validation, when the given form data is converted to an object of the type
`TYPO3\\CMS\\Extbase\\Mvc\\Controller\\Argument`.

### Manual validation

If you manually create a domain model object and want to make sure, that your Extbase validation rules are meet, you can
trigger the domain model validation manually as shown below:

{% highlight php %}
/** @var Data $dataModel */
$dataModel = $this->objectManager->get(Data::class);
$dataModel->setDescription('too short');

/* @var $validator \TYPO3\CMS\Extbase\Validation\Validator\ConjunctionValidator */
$validator = $this->objectManager->get(ValidatorResolver::class)->getBaseValidatorConjunction(Data::class);
$validationResults = $validator->validate($dataModel);

if ($validationResults->hasErrors()) {
    // @todo cycle through errors in $validationResults->getFlattenedErrors()
}
{% endhighlight %}

By creating the domain model object manually, you must take into account, that this will create a new object, where all
properties are initialized with the **default values** defined in the domain model.

### Practical use case

As an example for a practical use case for manual domain model validation, lets assume you have a REST webservice and
need to import some data to TYPO3. You typically fetch the data from the webservice and add the data to the database.
Instead of checking the content of the incoming record/field manually using if-statements, you can use the **Extbase
property mapper** in combination with **manual domain model validation**.

Below follows some example code for the described use case:

{% highlight php %}
/** @var PropertyMapper $propertyMapper */
$propertyMapper = $this->objectManager->get(PropertyMapper::class);

// Get some data - could for example be some data from a REST webservice
$data = $this->getApiData();

foreach ($data as $importRecord) {
    $dataModel = $propertyMapper->convert($importRecord, Data::class);
    // Note: The propertyMapper will set domain model default values for all all non-mappable values

    /* @var $validator \TYPO3\CMS\Extbase\Validation\Validator\ConjunctionValidator */
    $validator = $this->objectManager->get(ValidatorResolver::class)->getBaseValidatorConjunction(Data::class);
    $validationResults = $validator->validate($dataModel);

    if ($validationResults->hasErrors()) {
        // Record could not be imported, collect error messages for each field in $errorMessages array

        /** @var Error $error */
        foreach ($validationResults->getFlattenedErrors() as $field => $errors) {
            $errorMessages = [];
            foreach ($errors as $error) {
                $errorMessages[] = $error->getMessage();
            }
        }
    } else {
        // Import record to repository...
    }
}
{% endhighlight %}

By using the Extbase property mapper to create domain model objects, you do not need to check and assign each field
individually. You just have to make sure, that the array given passed to the "convert" function use the same field
naming as the domain model do like shown below.

{% highlight php %}
'title' => 'a title',
'email' => 'torben@derhansen.com',
'description' => 'a description',
'year' => 2017,
'amount' => 19.99,
'paid' => true
{% endhighlight %}

Note, that the resulting object from the property mapper will contain the **default values** for each property, that
can't be mapped properly.

In the code example above, the resulting domain model object will manually be validated and if no validation errors
occur, the object can be added to the repository.

I created a small [demo extension](https://github.com/derhansen/manual_domainvalidation) with a command controller,
which contains 2 example commands that show validation results for some dummy data.