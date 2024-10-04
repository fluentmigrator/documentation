---
uid: dotnet-fm
title: dotnet-fm tool
---

# The `FluentMigrator.MSBuild` tool

# Installation

The best way to install FluentMigrator.MSBuild is through GeneratePathProperty. However, care must be taken due to how MSBuild handles the evaluation phase.

# Running
To be set-up as part of your build process.

## .NET Runtime related switches

### `--allowDirtyAssemblies` switch

Overrides the default .NET Assembly Loading logic, allowing you to load assemblies written for other versions of the .NET runtime.
This is primarily intended for developers working on preview releases of the next version of the .NET runtime, but can be used by others as a workaround for assembly not found errors.  In particular, it is possible to have a "diamond dependency" where some dependencies have higher assembly versions than the one FluentMigrator uses.

For example, suppose FluentMigrator.DotNet.Cli ships with a particular version of System.ComponentModel.DataAnnotations.  Further suppose that you have a private enterprise nuget package repository that has re-usable FluentMigrator extension methods, which transitively references a different verison of System.ComponentModel.DataAnnotations.  There are only two possible ways you can call such an extension method: (1) implement your own FluentMigrator.DotNet.Cli tool and directly package your migrations DLL with the tool so that the MSBuild Microsoft.NET.Sdk correctly builds a project.deps.json used to resolve the right assembly (2) use this `--allowDirtyAssemblies` switch.

As another example, suppose FluentMigrator has not yet shipped a .NET vNext compatible binary, but you want to work with that binary.  The `--allowDirtyAssemblies` switch will help resolve the `System.Runtime` assembly.

### `UseMsBuildLogging` (optional, defaults to false)

Historically, the FluentMigrator.MSBuild task uses Console.Out to display output.  As MSBuild is multi-threaded, it is unsafe to do it this way.
Currently, this option only turns on diagnostic logging, but will be used in the future for logging all output.

## Connection related commands

### `Task` (required)

```
migrate               = Apply all migrations
migrate:up            = Apply all migrations
```

### `Target` aka Assembly (required)

The assembly containing the migrations you want to execute.

### `Connection` aka ConnectionString (required, can be a `connectionStringName`)

The connection string to the server and database you want to execute your migrations against. This can be a full connection string or the name of the connection string stored in a config file.

When specifying a named connection string, FluentMigrator searches for it in this order:

1. The specified config file via `--connectionStringConfigPath`](#cscp) parameter
2. Target assembly's config file
3. Machine.config config file

### `Timeout` (optional, specify value in whole seconds)

Overrides the default database command timeout of 30 seconds.

### `Profile` (optional)

The profile to run after executing migrations.

### `Tags` (optional)

Filters the migrations to be run by tags.

### `StripComments` (optional, defaults to false)

Whether comments should be stripped from SQL text prior to executing migration on server.
Default is true in 3.x; false will become the default in 4.x.