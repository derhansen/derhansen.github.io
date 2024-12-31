---
layout: post
title: 'Sorting by UIDs with MySQL FIELD function in TYPO3 13.4+ using Doctrine DBAL 4'
date: '2024-12-31T09:41.000+02:00'
author: Torben Hansen
tags:
- TYPO3
- doctrine
- ordering
- querybuilder
- extbase
- repository
modified_time: '2024-12-31T09:41.000+02:00'
permalink: /2024/12/sorting-by-uids-with-mysql-field-function-in-typo3-13-4-using-doctrine-dbal-4.html
---

Back in 2017, I wrote [this](https://www.derhansen.de/2017/05/mysql-field-function-in-typo3-87-with-doctrine.html) 
blogpost about how to use the MySQL `FIELD` function in TYPO3 with Doctrine DBAL. The function is used to apply
a **special ordering** of records by a given **list of uids** (e.g. from **FlexForm**) to a query using 
`QueryBuilder` and then uses the extbase datamapper to transform the raw Doctrine query result to an array of 
domain model objects.

Unfortunately, the solution does not work anymore in TYPO3 13.4+, since the `$queryBuilder->add()` function 
has been removed. To the rescue, it is still possible to apply the special ordering to the query using the `FIELD` 
function, but in a _different way_. 

Instead of using the `FIELD` function directly in the `ORDER BY` query part, it is now used as select literal to add 
the **current order index** as a **new column** to the query result. The new column is then used in the query via 
`$queryBuilder->orderBy()` as shown below.

```
<?php

declare(strict_types=1);

namespace MyVendor\MyExtension\Domain\Repository;

use Doctrine\DBAL\ArrayParameterType;
use TYPO3\CMS\Core\Database\ConnectionPool;
use TYPO3\CMS\Core\Utility\GeneralUtility;
use TYPO3\CMS\Extbase\Persistence\Generic\Mapper\DataMapper;
use TYPO3\CMS\Extbase\Persistence\Generic\Typo3QuerySettings;
use TYPO3\CMS\Extbase\Persistence\Repository;
use MyVendor\MyExtension\Domain\Model\MyModel;

class MyModelRepository extends Repository
{
    public function findByUidList(string $uidList = '')
    {
        $uids = GeneralUtility::intExplode(',', $uidList, true);
        if ($uids === []) {
            return [];
        }

        $dataMapper = GeneralUtility::makeInstance(DataMapper::class);
        $queryBuilder = GeneralUtility::makeInstance(ConnectionPool::class)
            ->getQueryBuilderForTable('tx_myext_domain_model_mymodel');

        $rows = $queryBuilder
            ->select('*')
            ->addSelectLiteral(
                sprintf('FIELD(tx_myext_domain_model_mymodel.uid, %s) AS custom_ordering', implode(',', $uids))
            )
            ->from('tx_myext_domain_model_mymodel')
            ->where(
                $queryBuilder->expr()->in(
                    'uid',
                    $queryBuilder->createNamedParameter(
                        $uids,
                        ArrayParameterType::INTEGER
                    )
                )
            )
            ->orderBy('custom_ordering')
            ->executeQuery()
            ->fetchAllAssociative();

        return $dataMapper->map(MyModel::class, $rows);
    }
}
```

The important part here is `addSelectLiteral`, where the MySQL `FIELD` function is used to add the new column
`custom_ordering` containing the ordering index. 

Note, that the additional column `custom_ordering` is **not taken into account** when the result is processed 
by `DataMapper`, because `DataMapper` only processes columns, which are accessible properties of the domain model.
