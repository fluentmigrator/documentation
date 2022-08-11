---
uid: maintenance-migrations.md
title: Maintenance Migrations
---

# Maintenance Migrations

## Overview
Maintenace Migration are a flexible way to apply logic on every migration run. They're flexible in that there are several "stages" the logic can be applied.

## Migration Stages

1. `BeforeAll`: Migration will be run before all standard migrations.
2. `BeforeEach`: Migration will be run before each standard migration.
3. `AfterEach`: Migration will be run after each standard migration.
4. `BeforeProfiles`: Migration will be run after all standard migrations, but before profiles.
5. `AfterAll`: Migration will be run after all standard migrations and profiles.    

## Use Cases

1. In the [FAQ](xref:faq), we demonstrate using Maintenance Migrations to create an "application lock" so that only one Migration Runner can run on a database at a time.
2. Seeding test/development data.
3. SQL Server-specific use cases that may potentially create ideas for users of other databases:
    1. Automatically refreshing SQL Server database views metadata are refreshed via [`sp_refreshview`](https://docs.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/sp-refreshview-transact-sql?view=sql-server-ver16)
    2. [Fixing the broken dependencies on all objects](https://github.com/ktaranov/sqlserver-kit/blob/master/Scripts/Fix_the_broken_dependencies_on_all__objects.sql) so that database replication works.


## Syntax

The following example will print `"Tag=[Development] Stage=[BeforeAll]"` when running the migration with `--tag=Development`.

```c#
using FluentMigrator;
using System;

namespace YourCompany.YourProduct.Database.Maintenance
{
    public class TagNames
    {
        public const string Development = nameof(Development);
    }

    [Tags(TagNames.Development)]
    [Maintenance(MigrationStage.BeforeAll, TransactionBehavior.None)]
    public class DevBeforeAll : ForwardOnlyMigration
    {
        public override void Up()
        {
            System.Console.WriteLine($"Tag=[{TagNames.Development}] Stage=[{MigrationStage.BeforeAll}]");
        }
    }
}
```
