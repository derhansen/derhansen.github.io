---
layout: post
title: Extbase $query->statement() - What can possibly go wrong?
date: '2019-04-01T14:20:00.000+02:00'
author: Torben Hansen
tags:
- wrong results
- repository
- Extbase
- datamapper
modified_time: '2019-04-01T15:29:33.704+02:00'
blogger_id: tag:blogger.com,1999:blog-6517038209122183182.post-2731958449882066714
blogger_orig_url: http://www.derhansen.de/2019/04/extbase-query-statement-what-can-possibly-go-wrong.html
permalink: /2019/04/extbase-query-statement-what-can-possibly-go-wrong.html
---

Last week I had to resolve a problem in a 3rd party Extension, where an Extbase Plugin returned unexpected results when
used _multiple times_ on the same page. The problem showed up in the frontend, where the plugin listed some products by
a given category. When the plugin was present one time on a page, the output was as following (simplified):

**Output of plugin 1**  
Product 1 for Category 1  
Product 2 for Category 1  
Product 3 for Category 1

When the plugin was present **two times** on a page, the output was as following (simplified):

**Output of plugin 1 with Category 1 as selection criteria**  
Product 1 for Category 1 (uid 1)  
Product 2 for Category 1 (uid 2)  
Product 3 for Category 1 (uid 3)

**Output of plugin 2 with** **Category 2 as selection criteria**  
Product 1 for Category 2 (uid 10)  
Product 2 for Category 1 (uid 2) <-- _**Whoops!**_  
Product 3 for Category 2 (uid 11)

Somehow, the output of plugin 2 contained a result, that did not belong to the result set. As written, the examples
above are simplified. The output on the production website showed hundreds of products, and just some of them were
wrong.

In order to debug the problem, I had a look at the Extbase Repository for the Products Domain model and found this (
again simplified).

{% highlight php %}
class ProductRepository extends \TYPO3\CMS\Extbase\Persistence\Repository
{
    /**
     * @param $categoryUid
     * @return array|\TYPO3\CMS\Extbase\Persistence\QueryResultInterface
     */
    public function findByCategory($categoryUid)
    {
        $query = $this->createQuery();
        $query->statement('SELECT * FROM tx_products_domain_model_product_' . $categoryUid);
        return $query->execute();
    }
}
{% endhighlight %}

OK... so there are several _individual tables_ for products by category. They all have the same structure and the only
difference is, that they have a different name (post-fixed with the category uid) and hold different data. There is also
a SQL injection vulnerability, but that has nothing to do with the main problem.

### What goes wrong here?

In order to explain, why plugin 2 returns an object, that obviously belongs to plugin 1, you have to know the internals
of an **Extbase repository,** the **Extbase QueryResult object** and the **DataMapper**.

Extbase determines the **Domain Model based on the Classname**. This is done in the constructor of the repository like
shown below:

{% highlight php %}
public function __construct(\TYPO3\CMS\Extbase\Object\ObjectManagerInterface $objectManager)
{
    $this->objectManager = $objectManager;
    $this->objectType = ClassNamingUtility::translateRepositoryNameToModelName($this->getRepositoryClassName());
}
{% endhighlight %}

So when the **findByCategory** function uses the **createQuery()** function, the query is initialized to create a query
for the **object type** the Repository determined (in this case Product).

When the query is executed using **$query-execute()**, it returns an object of the type 
`\\TYPO3\\CMS\\Extbase\\Persistence\\Generic\\QueryResult` and here we come closer to the explanation of the problem.
The QueryResult object has the following function:

{% highlight php %}
protected function initialize()
{
    if (!is_array($this->queryResult)) {
        $this->queryResult = $this->dataMapper->map(
            $this->query->getType(),
            $this->persistenceManager->getObjectDataByQuery($this->query)
        );
    }
}
{% endhighlight %}

This function uses the result from the persistenceManager (raw data from the database with language/workspace overlay)
and uses the **TYPO3 DataMapper** to create an array with Objects of the given type (Product). The DataMapper does this
row by row using the following function **mapSingleRow($className, array $row)**

And here is the final explanation for the behavior of the 2 plugins on the same page.

{% highlight php %}
protected function mapSingleRow($className, array $row)
{
    if ($this->persistenceSession->hasIdentifier($row['uid'], $className)) {
        $object = $this->persistenceSession->getObjectByIdentifier($row['uid'], $className);
    } else {
        $object = $this->createEmptyObject($className);
        $this->persistenceSession->registerObject($object, $row['uid']);
        $this->thawProperties($object, $row);
        $this->emitAfterMappingSingleRow($object);
        $object->_memorizeCleanState();
        $this->persistenceSession->registerReconstitutedEntity($object);
    }
    return $object;
}
{% endhighlight %}

For performance reasons, the **DataMapper caches all objects** it creates based on their UID. Since the repository in
this TYPO3 extension uses different tables (with own UIDs) for data storage, it may happen, that the **DataMapper
already processed** an object with the given UID (but from a different table) and therefore will return a cached version
of an object.

So when the output for plugin 1 was created, the DataMapper did create a **cached Product object for UID 2** and when
the output for plugin 2 was created, the DataMapper returned the **cached version** of the Product object with UID 2.

So always keep in mind, that an Extbase repository will return objects of exactly one type and that the datasource must
always contain unique uids.