---
uid: faq
title: FAQ
---

This FAQ will answer most common questions. Please [open an issue](https://github.com/fluentmigrator/fluentmigrator/issues) if your question isn't answered here.

## Why does the `Migrate.exe` tool say `No migrations found`?

Possible reasons:

- Migration class isn't public
- Migration class isn't derived from `IMigration` (or `Migration`)
- Migration class isn't attributed with `MigrationAttribute`
- The versions of the `Migrate.exe` tool (`FluentMigrator.Console` package) and the `FluentMigrator` package(s) referenced in your project are different.

## What are the supported databases?

[!include[Supported databases](../snippets/supported-databases.md)]

## How do I get the name of a SQL Server database?

Not all databases have a "name". Writing migrations that use a name therefore cannot be truly database-agnostic. That said, the following will show how to do it in FM2 and FM3:

### FM2
Use dynamic SQL:
```csharp
    public class GenerateDataMinutelyProfile : Migration
    {
        public override void Up()
        {
            this.Execute.Sql(@"
              DECLARE @DbName sysname = DB_NAME();
              DECLARE @SqlCommand NVARCHAR(MAX) = 'USE [master]; SET DEADLOCK_PRIORITY; ALTER DATABASE [' + @DbName + ']' + ' SET SINGLE_USER WITH ROLLBACK IMMEDIATE;';
              
              EXEC(@SqlCommand);
              SET @SqlCommand NVARCHAR(MAX) = 'USE [' + @DbName + ']';
              EXEC(@SqlCommand);
            ");
    }
```

### FM3

TODO

## How do I get the SQL Server file path and log path of the database, prior to running a migration?
