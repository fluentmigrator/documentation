---
uid: in-memory-testing
title: In-Memory Testing
---

# In-Memory Testing

When performing integration testing, it's sometimes helpful to be able to use an in-memory database (such as Sqlite) to act as the database while tests are running. As a result, this article describes how you can accomplish this with FluentMigrator.

_Credit to Craig on this [StackOverflow answer](https://stackoverflow.com/a/66926940) that in turn created an issue for us to add this to the documentation._

_Note: Sqlite only keeps the in-memory database in memory for the time the process is running. In addition, all connection strings that utilize the same name will access the same database._

Create InMemory database, setup FluentMigrator and execute migrations during test setup:

```csharp
public static IDisposable CreateInMemoryDatabase(Assembly AssemblyWithMigrations, IServiceCollection services = null)
{
    if (services == null)
        services = new ServiceCollection();

    var connectionString = GetSharedConnectionString();
    var connection = GetPersistantConnection(connectionString);
    MigrateDb(services, connectionString, AssemblyWithMigrations);

    services.AddSingleton<IConnectionFactory>(new InMemoryConnectionFactory(connectionString));

    return services.BuildServiceProvider()
        .GetRequiredService<IDisposableUnderlyingQueryingTool>();
}

private static string GetSharedConnectionString()
{
    var dbName = Guid.NewGuid().ToString();
    return $"Data Source={dbName};Mode=Memory;Cache=Shared";
}

private static void MigrateDb(IServiceCollection services, string connectionString, Assembly assemblyWithMigrations)
{
    var sp = services.AddFluentMigratorCore()
        .ConfigureRunner(fluentMigratorBuilder => fluentMigratorBuilder
            .AddSQLite()
            .WithGlobalConnectionString(connectionString)
            .ScanIn(assemblyWithMigrations).For.All() // Get all migrations, maintenance migrations and customizations
        )
        .BuildServiceProvider();

    var runner = sp.GetRequiredService<IMigrationRunner>();
    runner.MigrateUp();
}

private static IDbConnection GetPersistantConnection(string connectionString)
{
    var connection = new SqliteConnection(connectionString);
    connection.Open();

    return connection;
}
```

This serves as an example test utilizing the static methods above:

```csharp
[TestFixture]
public Test : IDisposable {
    private readonly IDisposable _holdingConnection;

    [Test]
    public Test() {
        _holdingConnection = CreateInMemoryDatabase(typeof(MyFirstMigration).Assembly);
    }

    public void Dispose() {
        _holdingConnection.Dispose();
    }
}
```
