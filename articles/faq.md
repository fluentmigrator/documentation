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
            /* Before you set the database to SINGLE_USER, verify that the AUTO_UPDATE_STATISTICS_ASYNC option is set to OFF. When this option is set to ON, the background thread that is used to update statistics takes a connection against the database, and you will be unable to access the database in single-user mode. For more information, see ALTER DATABASE SET Options (Transact-SQL). */
            this.Execute.Sql(@"
              DECLARE @DbName sysname = DB_NAME();
              DECLARE @SqlCommand NVARCHAR(MAX) = '
USE [master];
SET DEADLOCK_PRIORITY 10;
DECLARE @AutoUpdateStatisticsAsync BIT = CAST(0 AS BIT);
IF EXISTS (
    SELECT NULL
    FROM sys.databases WHERE name = @DbName AND is_auto_update_stats_async_on = CAST(1 AS BIT)
)
BEGIN
    ALTER DATABASE [' + @DbName + '] 
    SET AUTO_UPDATE_STATISTICS_ASYNC OFF;
    SET @AutoUpdateStatisticsAsync = CAST(1 AS BIT);
END;
ALTER DATABASE [' + @DbName + ']' + ' SET SINGLE_USER WITH ROLLBACK IMMEDIATE;
IF (@AutoUpdateStatisticsAsync = 1)
BEGIN
    ALTER DATABASE [' + @DbName + '] 
    SET AUTO_UPDATE_STATISTICS_ASYNC ON;
END;
';
              
              EXEC(@SqlCommand);
              SET @SqlCommand NVARCHAR(MAX) = 'USE [' + @DbName + ']';
              EXEC(@SqlCommand);
            ");
    }
```

### FM3

TODO

## How do I get the SQL Server file path and log path of the database, prior to running a migration?
