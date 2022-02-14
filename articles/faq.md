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

## How can I run FluentMigrator.DotNet.Cli with a .NET 5.0 assembly?

The FluentMigrator.DotNet.Cli contains an `--allowDirtyAssemblies` switch that will allow you to load your 5.0 assemblies in a .NET Core 3.1 context.  We're working on .NET 5.0 support.

## What are the supported databases?

[!include[Supported databases](../snippets/supported-databases.md)]

## How can I run migrations safely from multiple application servers?

Many server-side applications are load balanced and run multiple instances of the same application simultaneously from multiple web servers. In such a scenarios, if you choose to run migrations in-process (as opposed to an external migration runner), then there is an added risk of multiple processes trying to run migrations at the same time.

FluentMigrator does not automatically handle this scenario because the default transactional behavior is not enough to guarantee that only a single process can be running migrations at any given time. There are, however, some workarounds.

### Database-Dependent Application Locking

This style of solution depends upon MaintenanceMigrations. Two Maintenance Migrations are created: One with `BeforeAll` to atomically acquire a lock, and one `AfterAll` stage to release the lock.

This example is for **SQL Server 2008 and above** and uses [`sp_getapplock`](https://docs.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/sp-getapplock-transact-sql) to aquire a named lock before all migrations are run, and [`sp_releaseapplock`](https://docs.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/sp-releaseapplock-transact-sql) to release the lock after all migrations are finished.

```c#
[Maintenance(MigrationStage.BeforeAll, TransactionBehavior.None)]
public class DbMigrationLockBefore : Migration
{
    public override void Down()
    {
        throw new NotImplementedException("Down migrations are not supported for sp_getapplock");
    }

    public override void Up()
    {
        Execute.Sql(@"
            DECLARE @result INT
            EXEC @result = sp_getapplock 'MyApp', 'Exclusive', 'Session'

            IF @result < 0
            BEGIN
                DECLARE @msg NVARCHAR(1000) = 'Received error code ' + CAST(@result AS VARCHAR(10)) + ' from sp_getapplock during migrations';
                THROW 99999, @msg, 1;
            END
        ");
    }
}

[Maintenance(MigrationStage.AfterAll, TransactionBehavior.None)]
public class DbMigrationUnlockAfter : Migration
{
    public override void Down()
    {
        throw new NotImplementedException("Down migrations are not supported for sp_releaseapplock");
    }

    public override void Up()
    {
        Execute.Sql("EXEC sp_releaseapplock 'MyApp', 'Session'");
    }
}
```

In the above SQL Server example, we need to use `TransactionBehavior.None` on the Maintenance Migration while specifying the `@LockOwner` parameter to `Session`, which means that the locking behavior applies to the entire Session rather than a single transaction.

While the above is specific to SQL Server, similar concepts may available in other database providers.

* PostgreSQL has [Advisory Locks](https://www.postgresql.org/docs/10/explicit-locking.html#ADVISORY-LOCKS)
* SQL Anywhere has [Schema Locks](http://infocenter.sybase.com/help/topic/com.sybase.help.sqlanywhere.12.0.1/dbusage/transact-s-3443172.html)
* Oracle has [DBMS_LOCK.ALLOCATE_UNIQUE](https://docs.oracle.com/cd/A91202_01/901_doc/appdev.901/a89852/dbms_l2a.htm)
* DB2 has [LOCK TABLESPACE](https://www.ibm.com/support/knowledgecenter/en/SSEPEK_11.0.0/perf/src/tpc/db2z_lockmode.html) (with the caveat that every table in your migration is in the same tablespace)

### External Distributed Lock

If your database doesn't provide a means of acquiring an exclusive lock for migrations, it is still possible to achieve this functionality by using an external service for acquiring a distributed lock.

For example, Redis provides a way to perform [Distributed locks](https://redis.io/topics/distlock) so that different processes can operate on a shared resource in a mutually exclusive way. This scenario can be baked into a `BeforeAll`/`AfterAll` pair of Maintenance Migrations, as demonstrated above, to acquire an exclusive lock for the duration of the migration running.

As an alternative to Maintenance Migrations, which are by necessity specified in different classes, you could simply wrap the `MigrateUp()` call in code that acquires and releases the lock. Consider this pseudo-code which relies on [RedLock.net](https://github.com/samcook/RedLock.net):

```c#
async RunMigrationsWithDistributedLock(IMigrationRunner runner)
{
    var resource = "my-app-migrations";
    var expiry = TimeSpan.FromMinutes(5);

    using (var redLock = await redlockFactory.CreateLockAsync(resource, expiry)) // there are also non async Create() methods
    {
        // make sure we got the lock
        if (redLock.IsAcquired)
        {
            runner.MigrateUp();
        }
    }
    // the lock is automatically released at the end of the using block
}
```

### How can I execute a stored procedure using Oracle?

If you get `ORA-00900: Invalid SQL Statement` when executing a stored procedure, then chances are you need to wrap your stored procedure in a PLSQL block:

```c#
  Execute.Sql("DBMS_UTILITY.EXEC_DDL_STATEMENT('Create Index Member_AddrId On Member(AddrId)');");
```
becomes:
```c#
  Execute.Sql(@"
BEGIN
  DBMS_UTILITY.EXEC_DDL_STATEMENT('Create Index Member_AddrId On Member(AddrId)');
END;");
```

## How do I get the name of a SQL Server database?

Not all databases have a "name". Writing migrations that use a name therefore cannot be truly database-agnostic. That said, the following will show an example of getting the name so that you can perform an `ALTER DATABASE` command.  Note that to `ALTER DATABASE [YourDatabaseName]`, you need to switch to the `[master]` database first via `USE [master]`.  Then, since FluentMigrator does not call `sp_reset_connection`, you need to switch back to the database being migrated.  If you do not, the ensuing migrations will be run in the wrong database!

In the below example, we show how to enter a SQL Server database into single-user mode, in order to perform some maintenace tasks.

Use dynamic SQL:
```csharp
    public class EnterDatabaseSingleUserModeState : Migration
    {
        public override void Up()
        {
            /* Before you set the database to SINGLE_USER, verify that
            the AUTO_UPDATE_STATISTICS_ASYNC option is set to OFF. When
            this option is set to ON, the background thread that is
            used to update statistics takes a connection against the
            database, and you will be unable to access the database in
            single-user mode. For more information, see
            ALTER DATABASE SET Options (Transact-SQL). */
            this.Execute.Sql(@"
              DECLARE @DbName sysname = DB_NAME();
              DECLARE @SqlCommand NVARCHAR(MAX) = '
USE [master];
SET DEADLOCK_PRIORITY 10;
DECLARE @AutoUpdateStatisticsAsync BIT = CAST(0 AS BIT);
IF EXISTS (
    SELECT NULL
    FROM sys.databases WHERE name = @DbName AND is_auto_update_stats_async_on = CAST(1 AS BIT)
)
BEGIN
    ALTER DATABASE [' + @DbName + '] 
    SET AUTO_UPDATE_STATISTICS_ASYNC OFF;
    SET @AutoUpdateStatisticsAsync = CAST(1 AS BIT);
END;
ALTER DATABASE [' + @DbName + ']' + ' SET SINGLE_USER WITH ROLLBACK IMMEDIATE;

-- Here is where you put your administrative commands.

IF (@AutoUpdateStatisticsAsync = 1)
BEGIN
    ALTER DATABASE [' + @DbName + '] 
    SET AUTO_UPDATE_STATISTICS_ASYNC ON;
END;
';
              
              EXEC(@SqlCommand);
              SET @SqlCommand NVARCHAR(MAX) = 'USE [' + @DbName + ']';
              EXEC(@SqlCommand);
            ");
    }
```

## Certificate error for SQL Connection

Since the use of `Microsoft.Data.SqlClient` version `4.0.0` connections are encrypted by default in .NET. You might start getting the exception: `A connection was successfully established with the server, but then an error occurred during the login process. (provider: SSL Provider, error: 0 - The certificate chain was issued by an authority that is not trusted.)`. If fixing a valid certificate isn't a feasable path, you can always disable the encryption by adding `Encrypt=False` to the connection string. Then the connection won't be encryptet, and thus no certificate needed. As a note; you can also add `TrustServerCertificate=True` to the connection string, if you have a self-signed certificate or similar. But it's probably a better idea to fix a real certificate, or skip encryption all together.
