---
uid: dotnet-fm
title: dotnet-fm tool
---

# The `dotnet-fm` tool

# Installation

```bash
dotnet tool install -g FluentMigrator.DotNet.Cli
```

# Running

You can run the tool with `dotnet fm` or `dotnet-fm`. Use the latter version if the former doesn't work due to a .NET Core CLI tool bug.

# Command structure

```
dotnet-fm  -+- list -+- migrations   = List applied and pending migrations
            |        +- processors   = List all available processors
            +- migrate               = Apply all migrations
            |  +- up                 = Apply migrations up to given version (inclusive)
            |  +- down               = Apply migrations down to given version (exclusive)
            +- rollback              = Rollback the last migration applied
            |  +- all                = Rollback all migrations
            |  +- by                 = Rollback by <n> steps
            |  +- to                 = Rollback to given version
            +- validate --- versions = Validate order of applied migrations
```

# `list processors`

Shows all available processor identifiers to be used by the `-p`
or `--processor` command line switch.

## .NET Runtime related switches

### `--allowDirtyAssemblies` switch

Overrides the default .NET Assembly Loading logic, allowing you to load assemblies written for other versions of the .NET runtime.
This is primarily intended for developers working on preview releases of the next version of the .NET runtime, but can be used by others as a workaround for assembly not found errors.  In particular, it is possible to have a "diamond dependency" where some dependencies have higher assembly versions than the one FluentMigrator uses.

For example, suppose FluentMigrator.DotNet.Cli ships with a particular version of System.ComponentModel.DataAnnotations.  Further suppose that you have a private enterprise nuget package repository that has re-usable FluentMigrator extension methods, which transitively references a different verison of System.ComponentModel.DataAnnotations.  There are only two possible ways you can call such an extension method: (1) implement your own FluentMigrator.DotNet.Cli tool and directly package your migrations DLL with the tool so that the MSBuild Microsoft.NET.Sdk correctly builds a project.deps.json used to resolve the right assembly (2) use this `--allowDirtyAssemblies` switch.

As another example, suppose FluentMigrator has not yet shipped a .NET vNext compatible binary, but you want to work with that binary.  The `--allowDirtyAssemblies` switch will help resolve the `System.Runtime` assembly.

## Connection related commands

The following commands need a processor id and/or a connection:

- dotnet-fm list migrations
- dotnet-fm migrate
- dotnet-fm rollback
- dotnet-fm validate

### `-c|--connection <CONNECTION_STRING>`

The connection string itself to the server and database you want
to execute your migrations against.

### `--no-connection`

Indicates that migrations will be generated without consulting a target
database. Should only be used when generating an output file.

### `-p|--processor <PROCESSOR_NAME>`

The kind of database you are migrating against. Available choices can be
shown with [list processors](#list-processors).

### `-s|--processor-switches <PROCESSOR_SWITCHES>`

Database processor specific switches, e.g.:

- Firebird: `Force Quote=true`
- Oracle: `QUOTEDIDENTIFIERS=TRUE`

### `--preview`

Only output the migration steps - do not execute them. Add the `--verbose` switch to also see the SQL statements.
Default is false.

### `-V|--verbose`

Show the SQL statements generated and execution time in the console.
Default is false.

### `--profile <PROFILE>`

The profile to run after executing migrations.

### `--context <CONTEXT>` (obsolete)

Set ApplicationContext to the given string.

### `--timeout <TIMEOUT_SEC>`

Overrides the default database command timeout of 30 seconds.

### `-o|--output=<FILENAME>`

Output generated SQL to a file. Default is no output. A filename
may be specified, otherwise [targetAssemblyName].sql is the default.

### `-a|--assembly <ASSEMBLY_NAME>`

The assemblies containing the migrations you want to execute.

### `-n|--namespace <NAMESPACE>`

The namespace contains the migrations you want to run.
Default is all migrations found within the Target Assembly will be run.

### `--nested`

Whether migrations in nested namespaces should be included.
Used in conjunction with the namespace option.

### `--start-version`

The specific version to start migrating from.
Only used when NoConnection is true.
Default is 0.

### `--working-directory <WORKING_DIRECTORY>`

The directory to load SQL scripts specified by migrations from.

### `-t|--tag`

Filters the migrations to be run by tag.

### `-b|--allow-breaking-changes`

Allows execution of migrations marked as breaking changes.

# `rollback` and `migrate` specific parameters

### `-m|--transaction-mode <MODE>`

Overrides the transaction behavior of migrations, so that all
migrations to be executed will run in one transaction. Allowed
values are:

- Migration
- Session

# `rollback all`

Reverts all migrations.

# `rollback by <steps>`

Reverts the last `<steps>` migrations.

# `rollback to <version>`

Reverts all migrations down to (and excluding) `<version>`.

# `migrate up`

Applies the found migrations.

### `-t|--target <TARGET_VERSION>`

The specific version to migrate to (inclusive).

# `migrate down`

Applies the found migrations.

### `-t|--target <TARGET_VERSION>`

The specific version to revert to (exclusive).

### `--strip:<true|false>`

Whether comments should be stripped from SQL text prior to executing migration on server.
Default is true; false will become the default in 4.x.

# Examples

## List all available processors

```bash
dotnet fm list processors
```

## List applied and pending migrations

```bash
dotnet fm list migrations -p sqlite -c "Data Source=test.db" -a FluentMigrator.Example.Migrations.dll
```

## Apply all migrations

```bash
dotnet fm migrate -p sqlite -c "Data Source=test.db" -a FluentMigrator.Example.Migrations.dll
```

## Apply migrations up to given version (inclusive)

```bash
dotnet fm migrate -p sqlite -c "Data Source=test.db" -a FluentMigrator.Example.Migrations.dll up -t 20090906205342
```

## Apply migrations down to given version (exclusive)

```bash
dotnet fm migrate -p sqlite -c "Data Source=test.db" -a FluentMigrator.Example.Migrations.dll down -t 20090906205342
```

## Rollback the last migration applied

```bash
dotnet fm rollback -p sqlite -c "Data Source=test.db" -a FluentMigrator.Example.Migrations.dll
```

## Rollback to given version

```bash
dotnet fm rollback -p sqlite -c "Data Source=test.db" -a FluentMigrator.Example.Migrations.dll to 20090906205342
```

## Rollback by `<n>` steps

```bash
dotnet fm rollback -p sqlite -c "Data Source=test.db" -a FluentMigrator.Example.Migrations.dll by 1
```

## Rollback all migrations

```bash
dotnet fm rollback -p sqlite -c "Data Source=test.db" -a FluentMigrator.Example.Migrations.dll all
```

## Validate order of applied migrations

```bash
dotnet fm validate versions -p sqlite -c "Data Source=test.db" -a FluentMigrator.Example.Migrations.dll
```
