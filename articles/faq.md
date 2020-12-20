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
