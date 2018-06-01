---
uid: upgrade-guide-3.0-to-3.1
title: Upgrade Guide from 3.0 to 3.1
---

# Upgrading from 3.0 to 3.1

This version contains mostly added features and fixed bugs.

# What is new?

The `WithMigrationsIn` was well-meant, but not sufficient. We now
provide an additional way to scan assemblies for the interesting
stuff.

## New way to scan assemblies

```cs
// Initialize you FluentMigrator (and other) services
IServiceCollection services = ...;
services
    .ConfigureRunner(rb => rb
        // Define the assembly containing the migrations
        .ScanIn(typeof(AddLogTable).Assembly).For.Migrations());
```

The interesting part here is the new `ScanIn` function which allows
one to specify which assemblies should be scanned for the following
parts:

- Migrations
- Version table metadata
- Embedded resources
- All of the parts above

One `ScanIn` can be used to target multiple parts:

```cs
// Initialize you FluentMigrator (and other) services
IServiceCollection services = ...;
services
    .ConfigureRunner(rb => rb
        // Define the assembly containing the migrations
        .ScanIn(typeof(AddLogTable).Assembly)
            .For.Migrations()
            .For.EmbeddedResources());
```

It is also possible to use the assembly for everything:

```cs
// Initialize you FluentMigrator (and other) services
IServiceCollection services = ...;
services
    .ConfigureRunner(rb => rb
        // Define the assembly containing the migrations
        .ScanIn(typeof(AddLogTable).Assembly).For.All());
```

### Summary

This allows explicit configuration of the scanned assemblies
and avoids manual registration of an `IAssemblySourceItem` service, like this:

```cs
services.AddSingleton<IAssemblySourceItem>(() => new AssemblySourceItem(GetType().Assembly));
```

## `IFilteringMigrationSource`

This interface allows filtering of all migration types without the need to instantiate them first.

## `IVersionTableMetaDataSourceItem`

A new interface to specify multiple places to search for version table metadata.

# What has changed?

- [#884](https://github.com/fluentmigrator/fluentmigrator/issues/884): Embedded script cannot be found in assemblies on .NET Core
- [#888](https://github.com/fluentmigrator/fluentmigrator/issues/888): VersionTable not changed after upgrading to 3.0
- `IEmbeddedResourceProvider` can be registered multiple times (required for the new `ScanIn` feature)
- Query `IConfigurationRoot` for the connection string if `IConfiguration` couldn't be found
- `dotnet-fm` now uses the Oracle beta ADO.NET driver

# Oracle Beta ADO.NET driver

Oracle plans to release a non-beta version of the driver in Q3, but
it's the only Oracle driver that works under Linux/MacOS. The console
tool (`Migrate.exe`) is more Windows-centric and will therefore keep
using the standard Oracle ADO.NET library. The `dotnet-fm` is mostly
used on non-Windows platforms and is therefore predestinated to use
the new beta driver.

The statement from Oracle can be found on the
[Oracle website](http://www.oracle.com/technetwork/topics/dotnet/tech-info/odpnet-dotnet-ef-core-sod-4395108.pdf).

The console tool will switch to the new driver when it becomes stable.

# Dependency injection related changes

Some services are now registered as "Scoped" to allow the reconfiguration
of the connection string/used database at run-time.

## Applies to

- Connection string
- Processor/generator selection
- Type filters

## Changes

The following option classes are now resolved using `IOptionSnapshot<T>`:

- `ProcessorOptions`
- `SelectingProcessorAccessorOptions`
- `SelectingGeneratorAccessorOptions`
- `TypeFilterOptions`

The following services are now scoped instead of singleton:

- `IVersionTableMetaDataAccessor`
- `IVersionTableMetaData`
- `IMigrationSource`
- `IMigrationInformationLoader`

The `MigrationSource` now consumes all registered `IMigrationSourceItem` instances.

# What is fixed?

- [#877](https://github.com/fluentmigrator/fluentmigrator/issues/877): Connection specific information should be resolved as scoped
    - Also: [#882](https://github.com/fluentmigrator/fluentmigrator/issues/882)
- [#886](https://github.com/fluentmigrator/fluentmigrator/issues/886): Using profiles in 3.x versions
- [#892](https://github.com/fluentmigrator/fluentmigrator/issues/892): Nullable types are not supported in MSBuild runner
