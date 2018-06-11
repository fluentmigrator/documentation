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
