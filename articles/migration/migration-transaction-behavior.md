---
uid: migration-transaction-behavior
title: Migration transaction behavior
---

# Transaction modes for the migration runner

There are two modes, Transaction-Per-Session and Transaction-Per-Migration. The default mode for FluentMigrator is Transaction-Per-Migration.

## Transaction-Per-Migration ##

This mode means that each migration is run in its own transaction and allows selected migrations to be run with no transaction. This allows migrations to be able to do tasks like creating full-text indexes which cannot be run in a transaction.

This mode can be used in combination with the new [TransactionBehavior](xref:FluentMigrator.TransactionBehavior) attribute.

```cs
[Migration(201304210958, TransactionBehavior.None)]
```

For this migration it will be run outside of a transaction. The default value for [TransactionBehavior](xref:FluentMigrator.TransactionBehavior) is [Default](xref:FluentMigrator.TransactionBehavior.Default) which means the migration will be run in a transaction.

> [!NOTE]
> Not all databases support "Transactional Data Definition Language" (Transactional DDL).
> 
> * Sybase and SQL Server support some version of Transactional DDL through Transact-SQL language, but there are some limitations around schema binding, user-defined types, and when "names" are bound/resolved.
> * MySQL introducd Transactional DDL support in MySQL 8.0 as "[atomic DDL](https://dev.mysql.com/doc/refman/8.0/en/atomic-ddl.html#:~:text=MySQL%208.0%20supports%20atomic%20Data,into%20a%20single%2C%20atomic%20operation.)".

## Transaction-Per-Session ##

Up to this release, there has been one mode, Transaction-Per-Session. This means that if two migrations or more migrations are run together during the same task (e.g. migrate up or down) then this occurs in the same transaction and if one of the migrations fails then all the migrations are rolled back.

Transaction-Per-Session is the old default and if you want to use this mode then there are switches for the Command-Line runner, MSBuild runner, or .NET CLI tool to enable it.
