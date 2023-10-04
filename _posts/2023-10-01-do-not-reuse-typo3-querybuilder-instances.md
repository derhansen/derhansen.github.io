---
layout: post
title: 'The pitfalls of reusing TYPO3 QueryBuilder: Analyzing a performance bottleneck'
date: '2023-10-03T13:45:00.000+02:00'
author: Torben Hansen
tags:
- typo3
- querybuilder
- slow
- xdebug
- profiling
modified_time: '2023-10-04T07:38:00.000+02:00'
permalink: /2023/10/the-pitfalls-of-reusing-typo3-querybuilder-analyzing-a-performance-bottleneck.html
---

**TL;DR:** Do **never** reuse an instance of the TYPO3 `QueryBuilder` for queries, even if the query is 
the same but with _different parameters_, since this causes a significant **performance decreasement** when 
processing larger amount of records.

I was recently involved in refactoring an older codebase in a TYPO3 project. One part of that codebase was a 
data import which was created using a Symfony console command. The command basically fetched data from an 
external data source and imported/updated existing records in the TYPO3 database. During execution, I noticed 
that it was quite slow. What's more, it appeared that the routine slowed down as more records were processed, 
indicating a bottleneck in the code. After some investigation, it became evident that the slow performance was 
caused by the TYPO3 `QueryBuilder`, which was instantiated globally in the Symfony command and then reused in 
several functions.

The TYPO3 documentation advises against reusing the `QueryBuilder`, emphasizing that it holds an internal state.

![QueryBuilder note](/assets/images/2023-10-01/typo3-querybuilder-notice.png)

The notice does however not describe the _consequences_ of reusing a `QueryBuilder` instance. Since it caused
a significant performance decreasement in the code I was working on, I was curious and digged deeper into the 
topic to analyze, why `QueryBuilder` slowed down so much when reused for multiple queries.

### Performance analysis

To highlight the performance decrease, I created a simple Symfony command that:

1. Selected 1000 records from the `tx_extensionmanager_domain_model_extension` table.
2. Iterated through the query results.
3. Updated a field for each of the 1000 records.   

The command is executed once using a global instance of the `QueryBuilder` for all queries and once using a
new instance of the `QueryBuilder` for each query.

The code for updating looked like this:

```
$queryBuilder
    ->update('tx_extensionmanager_domain_model_extension')
    ->set('last_updated', (string)time())
    ->where(
        $this->queryBuilder->expr()->in(
            'uid',
            $this->queryBuilder->createNamedParameter($uid, Connection::PARAM_INT)
        )
    )
    ->executeStatement();
```

Note, that the `QueryBuilder` object in the code above should be assumed as either a global or a new instance.

#### Command execution time

* Global `QueryBuilder` instance for all queries: **7.8010120391846** seconds
* New `QueryBuilder` instance per query: **2.3586421012878** seconds

This clearly demonstrates that reusing an instance of the `QueryBuilder` for multiple queries results in a 
significant performance decrease.

### Profiling with xdebug and IntelliJ IDEA

To gain insights into the code execution, I used the Xdebug [profiler](https://xdebug.org/docs/profiler) to save 
profiling information to a file. Next, I used the "Xdebug profile analyzer" tool in IntelliJ 
(see [link](https://www.jetbrains.com/help/phpstorm/profiling-with-xdebug.html#analyze-xdebug-profiling-data)) to 
visualize the profiling results.

![QueryBuilder profiling](/assets/images/2023-10-01/profiling-querybuilder-1.png)

The screenshot reveals, that the execution of `Doctrine\DBAL\Connection->expandArrayParameters` and 
`Doctrine\DBAL\SQL\Parser->parse` consume a significant percentage of the overall processing time and that memory 
usage is notably high.

When navigating through the call tree of the process, it became evident that the initial update statement performed 
well (8 ms execution time):

![QueryBuilder call tree first update](/assets/images/2023-10-01/profiling-querybuilder-execute-1.png)

However, the last update statement was considerably slower, taking 81 ms compared to the first one: 

![QueryBuilder call tree last update](/assets/images/2023-10-01/profiling-querybuilder-execute-1000.png)

So as a result, the profiling uncovered, that the `Doctrine\DBAL\Connection->expandArrayParameters` calls should
be observed. To do so, I set breakpoints for the first and last iterations and analyzed the data being processed. 
The result for the first update call aligned with expectations:

![expandArrayParameters first call](/assets/images/2023-10-01/expand-array-parameters-1.png)

Here we have the SQL for the query and the 2 parameters which are used to replace the placeholder values in the query.

However, the result for the last update call finally revealed why reusing the `QueryBuilder` for multiple queries
is a bad idea:

![expandArrayParameters first call](/assets/images/2023-10-01/expand-array-parameters-1000.png)

When `QueryBuilder` is reused, the first update query is **extended** with each loop iteration, resulting in an 
excessively large SQL query containing 2000 parameters to parse and process. This explains the progressively slower 
performance with each iteration and the high memory consumption. Doctrine internally uses the PHP function 
`preg_match` to replace placeholder values, and as the number of placeholders increases, the replacement process 
becomes slower due to the overhead of parsing and processing a large number of patterns.

### Conclusion

As a result of the analysis, I think it may be good to change the notice in the documentation to a warning 
and outline, that you **must never** reuse an instance of the `QueryBuilder`.

_Update 04.10.2023:_ Thanks a lot to the TYPO3 documentation team, who already updated 
the `QueryBuilder` [documentation](https://docs.typo3.org/m/typo3/reference-coreapi/main/en-us/ApiOverview/Database/QueryBuilder/Index.html#instantiation) 
as suggested!