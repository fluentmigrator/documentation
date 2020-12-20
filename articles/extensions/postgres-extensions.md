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


Note: This feature was implement on Postgres 11, with you want to use it is necessary use Postgres11_0Generator.

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

This extension allows you to choose a specific index method (B-tree (default), hash, GiST, SP-GiST, GIN, and BRIN) for more information about index method see [here](https://www.postgresql.org/docs/current/indexes-types.html)

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