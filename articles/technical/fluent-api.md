---
uid: tech-fluent-api
title: Fluent API
---

# How does the fluent API work?

The starting points for the fluent API are the following classes:

- [MigrationBase](xref:FluentMigrator.MigrationBase): database modeling
- [Migration](xref:FluentMigrator.Migration): data modification and script execution

# Supported expressions

## [Create](xref:tech-fluent-api-create)

Create a database object.

## [Alter](xref:tech-fluent-api-alter)

Alter a database object.

## [Delete](xref:tech-fluent-api-delete)

Delete a database object or data.

## Rename ([IRenameExpressionRoot](xref:FluentMigrator.Builders.Rename.IRenameExpressionRoot))

Rename a database object.

## Schema ([ISchemaExpressionRoot](xref:FluentMigrator.Builders.Schema.ISchemaExpressionRoot))

Schema-oriented starting point for expressions.

## IfDatabase ([IIfDatabaseExpressionRoot](xref:FluentMigrator.Builders.IfDatabase.IIfDatabaseExpressionRoot))

Create database-specific expressions.

## Insert ([IInsertExpressionRoot](xref:FluentMigrator.Builders.Insert.IInsertExpressionRoot))

Insert data.

## Update ([IUpdateExpressionRoot](xref:FluentMigrator.Builders.Update.IUpdateExpressionRoot))

Update data.

## Execute ([IExecuteExpressionRoot](xref:FluentMigrator.Builders.Execute.IExecuteExpressionRoot))

Execute SQL statements/scripts.

# Planned changes

- Differentiate between `Delete` and `Drop`

