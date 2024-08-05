---
uid: migration-filter-tags
title: Fluent Interface
---

# Filter migrations run based on Tags

Scenario: multiple database once had the same schema but over the years have had differences introduced.

Because of this the majority of migrations, contained in one assembly, can be run against all the databases but at the same time you need to be able to filter certain migrations to run against specific databases based on the country (UK, DK, etc) and environment (Staging, Production, etc) it serves.

To facilitate this you can tag migrations (much like cucumber scenarios) and then filter these by passing tags into the runner. A migration is then run if it has:

* No Tags
* OR Has Tags that match ALL those passed into the runner.

Tags are assigned to migrations with an attribute/s:

```cs
[Tags("DK", "NL", "UK")]
[Tags("Staging", "Production")]
[Migration(1)]
public class DoSomeStuffToEuropeanStagingAndProdDbs : Migration { /* ... etc ... */ }
```

# Tag Matching Behavior

By default, FluentMigrator only runs migrations that have no tags or whose tags match all of the tags passed in to the runner.  Another option that can be used is passing in the `TagBehavior.Any` enum to the `[Tags]` attribute so that the migration will run if any of the tags defined in the attribute match those passed into the runner.

```cs
[Tags(TagBehavior.Any, "UK")]
[Migration(2)]
public class DoSomethingToUnitedKingdomDbs : Migration { /* ... etc ... */ }
```

# Inheriting Migration Tags

Tag inheritance can apply to both base classes (single-inheritance) and interfaces (multiple-inheritance).

Scenario: If you have a multiple contexts for which changes need to be applied, it can be less error-prone to inherit tags from a base class.

```cs
// Single-Inheritance of a base class
[Tags("Development", "QA", "UAT")]
public class NonProductionMigration : Migration { /* ... etc ... */ }

public class ApplySomeNonProductionChanges : NonProductionMigration { /* ... etc ... */ }
```

Scenario: This is especially the case if you have common +AND+ divergent functionality that needs to be applied to multiple contexts. In this particular case, it is less error-prone to apply tags to one or more interfaces that may be combined. Consider the case of geographical database segregation whereby some functionality is applied to one or more instances based on geographical requirements.

```cs
// tagged interfaces
[Tags("AL", "TX")]
public interface IApplyToFeature1 { /* ... etc ... */ }

[Tags("CA", "NV")]
public interface IApplyToFeature2 { /* ... etc ... */ }

[Tags("AL", "NV")]
public interface IApplyToFeature3 { /* ... etc ... */ }

[Tags("TX")]
public interface IApplyToFeatureWithSynchronousResponse { /* ... etc ... */ }

[Tags("AL", "NV")]
public interface IApplyToFeatureWithAsynchronousQueueResponse { /* ... etc ... */ }

[Tags("CA")]
public interface IApplyToFeatureWithAsynchronousBatchResponse { /* ... etc ... */ }

// Inheritance of interfaces
public interface IApplyToFeatureWithAsynchronousResponses : IApplyToFeatureWithAsynchronousBatchResponse, IApplyToFeatureWithAsynchronousQueueResponse { /* ... etc ... */ }

public class ApplySomeChangeToFeature1 : Migration, IApplyToFeature1 { /* ... etc ... */ }

public class ApplySomeChangeToFeatures2And3 : Migration, IApplyToFeature2, IApplyToFeature3 { /* ... etc ... */ }

public class ApplySomeChangeToFeatureWithAsynchronousResponses : Migration, IApplyToFeatureWithAsynchronousResponses { /* ... etc ... */ }
```

This allows for a Dev Ops pipeline that requires migrations based on a tag (in the case above, by state), and thus simplifying how to discern which migrations are applied based on that tag. This also means that creating a new database context for a state is a matter of deciding which features the new tag is associated with, and updating those relevant interfaces rather than having to add tags to all of the relevant migrations.

# Filtering

## [`IMigrationRunner`](#tab/runner-internal)

```cs
var serviceProvider = new ServiceCollection()
    .AddFluentMigratorCore()
    .ConfigureRunner(rb => rb
        .AddSqlServer2008()
        .WithGlobalConnectionString("server=.\\SQLEXPRESS;uid=testfm;pwd=test;Trusted_Connection=yes;database=FluentMigrator")
        .ScanIn(typeof(DoSomeStuffToEuropeanStagingAndProdDbs).Assembly).For.Migrations())
    .AddLogging(lb => lb.AddFluentMigratorConsole())
    // Start of type filter configuration
    .Configure<RunnerOptions>(opt => {
        opt.Tags = new[] { "UK", "Production" }
    })
    .BuildServiceProvider(false);

var runner = serviceProvider.GetRequiredService<IMigrationRunner>();
runner.MigrateUp();
```

## [`Migrate.exe` tool](#tab/runner-external-migrate)

And the tags are entered into the command line like so: (Migrations with UK AND Production tags executed)

<pre><code><migrate.exe command> --tag UK --tag Production</code></pre>

## [MSBuild task](#tab/runner-external-msbuid)

For the Msbuild runner, there is a Tags option and tags are passed in as a comma-separated string.

```xml
<Migrate Database="sqlserver2008"
    Connection="server=.\SQLEXPRESS;uid=testfm;pwd=test;Trusted_Connection=yes;database=FluentMigrator"
	Target="Migrations" 
    Tags="UK,Production" />
```

***
