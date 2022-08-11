---
uid: migration-constraints.md
title: Migration Constraints
---

# Migration Constraints

## Overview

Migration constraints allow users to run migrations based on dynamically evaluated conditions.

## Use Case

The `FluentMigrator.Runner.Core` namespace has an example, `CurrentVersionMigrationConstraintAttribute`.

## Examples

The FluentMigrator samples folder has a demonstration of this example:

```c#
using FluentMigrator.Runner.Constraints;

namespace FluentMigrator.Example.Migrations
{
    /// <summary>
    /// Update notes.
    /// </summary>
    /// <remarks>
    /// This is only run when the migration 20090906205440 was already applied to the database.
    /// </remarks>
    [Migration(20190416112000)]
    [CurrentVersionMigrationConstraint(20090906205440)]
    public class UpdateNotesByScript : Migration
    {
        /// <inheritdoc />
        public override void Up()
        {
            Execute.Sql(@"/* this is a test script */
update Notes set body=body || ' (modified)';
");
        }

        /// <inheritdoc />
        public override void Down()
        {
            // Nothing to do here
        }
    }
}
```

## Caveats

The downside to migration constraints, and `CurrentVersionMigrationConstraintAttribute  in particular, is that some data is only loaded by FluentMigrator once.  For example, version metadata is snapshot once.  This is because FluentMigrator builds a list of all migrations to run, once, and then runs them.  Thus, you can imagine a scenario where the above example is run, and during the migration run, version 20090906205440 executes.  However, UpdateNotesByScript won't execute, because when the runner initializes, 20090906205440 wasn't yet applied to the database.