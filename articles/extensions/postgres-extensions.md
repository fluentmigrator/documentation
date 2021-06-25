---
uid: postgres-extensions.md
title: Postgres specific extensions
---

FluentMigrator supports some extra functions that are specific to Postgres only.

# Adding the FluentMigrator.Runner assembly as a reference

These extension methods are not included in the core dll so to get access them you have to add the FluentMigrator.Extensions.Postgres package in your migrations project. The final step is to add the following using to your migration class:

```cs
using FluentMigrator.Postgres;
```
## Include column on index

This extension allows you to include a column for an index.


```cs
Create.Index()
    .OnTable("TestTable")
    .Column("Id").Ascending()
    .Include("Name");
```


Note: This feature was implement on Postgres 11, with you want to use it is necessary use Postgres11_0 or higher.

```cs
 var service = new ServiceCollection()
                // Add common FluentMigrator services
                .AddFluentMigratorCore()
                .ConfigureRunner(rb => rb
                    // Add Postgres 11 support to FluentMigrator
                    .AddPostgres11_0()
                    ..
                );
```

## Index method

This extension allows you to choose a specific index method (B-tree (default), Hash, GiST, SP-GiST, GIN, and BRIN) for more information about index method see [here](https://www.postgresql.org/docs/current/indexes-types.html)

```cs
Create.Index()
    .OnTable("TestTable")
    .Column("Id").Ascending()
    .WithOptions()
    .UsingHash();
```
Or if you want more control over index method you could pass the index method by parameter:

```cs
Create.Index()
    .OnTable("TestTable")
    .Column("Id").Ascending()
    .WithOptions()
    .Using(Algorithm.Gin);
```

Notes: Dependend on the index method could or couldn't support some feature, for sample Hash index method doesn't support multi-column. For more information about index method limitation see [here](https://www.postgresql.org/docs/current/sql-createindex.html) in Notes parts.


## Concurrently

When this option is used, PostgreSQL will build the index without taking any locks that prevent concurrent inserts, updates, or deletes on the table. Whereas a standard index build locks out writes (but not reads) on the table until it's done. For more information about index method see [here](https://www.postgresql.org/docs/current/sql-createindex.html#SQL-CREATEINDEX-CONCURRENTLY).


```cs
Create.Index()
    .OnTable("TestTable")
    .Column("Id").Ascending()
    .WithOptions()
    .AsConcurrently();
```
### Notes
For to be able to use this feature is necessary remove the transaction from migrations, because PostgeSQLd doesn't support create concurrently index with transaction.

## Only

Indicates not to recurse creating indexes on partitions, if the table is partitioned

```cs
Create.Index()
    .OnTable("TestTable")
    .Column("Id").Ascending()
    .WithOptions()
    .AsOnly();
```

## Null sorts

Specifies that column should be sort before or after non-nulls. 

```cs
Create.Index()
    .OnTable("TestTable")
    .Column("Id").Ascending().Nulls(NullSort.First);
```

### Nulls First

Specifies that nulls sort before non-nulls. This is the default when DESC is specified.
```cs
Create.Index()
    .OnTable("TestTable")
    .Column("Id").Ascending().NullsFirst()
```

### Nulls Last

Specifies that nulls sort after non-nulls. This is the default when DESC is not specified.

```cs
Create.Index()
    .OnTable("TestTable")
    .Column("Id").Ascending().NullsLast()
```

## Partial Indexes (WHERE/Filter)

A partial index is an index built over a subset of a table; the subset is defined by a conditional expression (called the predicate of the partial index). The index contains entries only for those table rows that satisfy the predicate. For more information about index method see [here](https://www.postgresql.org/docs/current/indexes-partial.html).


```cs
Create.Index()
    .OnTable("TestTable")
    .Column("Id").Ascending()
    .Column("is_enable").Ascending()
    .WithOptions()
    .Filter("is_enable is true");
```

## Index Storage Parameters

Each index method has its own set of allowed storage parameters. For more information about index storageparameters see [here](https://www.postgresql.org/docs/current/sql-createindex.html#SQL-CREATEINDEX-STORAGE-PARAMETERS)


### Fillfactor

The fillfactor for an index is a percentage that determines how full the index method will try to pack index pages. For B-trees, leaf pages are filled to this percentage during initial index build, and also when extending the index at the right (adding new largest key values). If pages subsequently become completely full, they will be split, leading to gradual degradation in the index's efficiency. B-trees use a default fillfactor of 90, but any integer value from 10 to 100 can be selected. If the table is static then fillfactor 100 is best to minimize the index's physical size, but for heavily updated tables a smaller fillfactor is better to minimize the need for page splits. The other index methods use fillfactor in different but roughly analogous ways; the default fillfactor varies between methods.


```cs
Create.Index()
    .OnTable("TestTable")
    .Column("Id").Ascending()
    .WithOptions()
    .Fillfactor(100);
```

Notes: In case you use Fillfactor without have select an index method we are going to assume that you are using B-Tree (the defautl index method) and the release B-Tree index storage.

### B-Tree

The exclusive index storage parameters for B-Tree index method.

### Vacuum cleanup index scale factor

Specifies the fraction of the total number of heap tuples counted in the previous statistics collection that can be inserted without incurring an index scan at the VACUUM cleanup stage. The value can range from 0 to 10_000_000_000. When vacuum_cleanup_index_scale_factor is set to 0, index scans are never skipped during VACUUM cleanup. The default value is 0.1. 

For more information aboutVacuum cleanup index scale factor see [here](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-VACUUM-CLEANUP-INDEX-SCALE-FACTOR)

```cs
Create.Index()
    .OnTable("TestTable")
    .Column("Id").Ascending()
    .WithOptions()
    .UsingBTree() 
    .VacuumCleanupIndexScaleFactor(12.0f);
```

Note: This feature was implement on Postgres 11, if you want to use it is necessary use Postgres11_0 or higher.

### GiST

The exclusive index storage parameters for GiST index method.

### Buffering

Building large GiST indexes by simply inserting all the tuples tends to be slow, because if the index tuples are scattered across the index and the index is large enough to not fit in cache, the insertions need to perform a lot of random I/O. Beginning in version 9.2, PostgreSQL supports a more efficient method to build GiST indexes based on buffering, which can dramatically reduce the number of random I/Os needed for non-ordered data sets. For well-ordered data sets the benefit is smaller or non-existent, because only a small number of pages receive new tuples at a time, and those pages fit in cache even if the index as whole does not.

For more information about Buffering see [here](https://www.postgresql.org/docs/current/gist-implementation.html#GIST-BUFFERING-BUILD)

```cs
Create.Index()
    .OnTable("TestTable")
    .Column("Id").Ascending()
    .WithOptions()
    .UsingGist()
    .Buffering(GistBuffering.Auto)
```

Note: This feature was implement on Postgres 10, if you want to use it is necessary use Postgres10_0.


### GIN

The exclusive index storage parameters for GIN index method.

### Fastupdate

Updating a GIN index tends to be slow because of the intrinsic nature of inverted indexes: inserting or updating one heap row can cause many inserts into the index (one for each key extracted from the indexed item). Because of that nature of GIN index, PostgreSQL add support for Fastupdate,  where that is capable of postponing much of this work by inserting new tuples into a temporary, unsorted list of pending entries. 

For more information about Fastupdate see [here](https://www.postgresql.org/docs/current/gin-implementation.html#GIN-FAST-UPDATE)

```cs
Create.Index()
    .OnTable("TestTable")
    .Column("Id").Ascending()
    .WithOptions()
    .UsingGin()
    .FastUpdate()
```

```cs
Create.Index()
    .OnTable("TestTable")
    .Column("Id").Ascending()
    .WithOptions()
    .UsingGin()
    .DisableFastUpdate()
```

### Pending list limit

Sets the maximum size of a GIN index's pending list, which is used when fastupdate is enabled. If the list grows larger than this maximum size, it is cleaned up by moving the entries in it to the index's main GIN data structure in bulk. If this value is specified without units, it is taken as kilobytes. The default is four megabytes (4MB).

For more information about Pending list limit see [here](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-GIN-PENDING-LIST-LIMIT)

```cs
Create.Index()
    .OnTable("TestTable")
    .Column("Id").Ascending()
    .WithOptions()
    .UsingGin()
    .PendingListLimit(1_000)
```

Note: This feature was implement on Postgres 10, if you want to use it is necessary use Postgres10_0 or higher.

### BRIN

The exclusive index storage parameters for BRIN index method.


### Pages per range

Defines the number of table blocks that make up one block range for each entry of a BRIN index. The default is 128.

For more information about BRIN index see [here](https://www.postgresql.org/docs/current/brin-intro.html)

```cs
Create.Index()
    .OnTable("TestTable")
    .Column("Id").Ascending()
    .WithOptions()
    .UsingBrin()
    .PagesPerRange(127)
```

Note: This feature was implement on Postgres 10, if you want to use it is necessary use Postgres10_0 or higher.

### Autosummarize

Defines whether a summarization run is invoked for the previous page range whenever an insertion is detected on the next one.

For more information about BRIN index see [here](https://www.postgresql.org/docs/current/brin-intro.html)

```cs
Create.Index()
    .OnTable("TestTable")
    .Column("Id").Ascending()
    .WithOptions()
    .UsingBrin()
    .Autosummarize()
```

```cs
Create.Index()
    .OnTable("TestTable")
    .Column("Id").Ascending()
    .WithOptions()
    .UsingBrin()
    .DisableAutosummarize()
```

Note: This feature was implement on Postgres 10, if you want to use it is necessary use Postgres10_0 or higher.