---
uid: sql-server-extensions.md
title: SQL Server specific extensions
---

FluentMigrator supports some extra functions that are specific to Sql Server only.

# Adding the FluentMigrator.Runner assembly as a reference

These extension methods are not included in the core dll so to get access them you have to add the FluentMigrator.Extensions.SqlServer package in your migrations project. The final step is to add the following using to your migration class:

```cs
using FluentMigrator.SqlServer;
```
## Constraint Clustering (Clustered or NonClustered)

This extension allows you to specify a primary key or unique constraint as clustered or non-clustered.
SQL Server tries do the following, unless you specify 'clustered' or 'nonclustered' in sql:

* Create a primary key with a 'clustered' index
* Create a unique constraint with a 'nonclustered' index

Therefore, the most common usage pattern is likely to be along the lines of:

### Create a primary key with a 'nonclustered' index

```cs
Create.PrimaryKey("PK").OnTable("TestTable").Column("Id").NonClustered();
```

### Create a unique constraint with a 'clustered' index

```cs
Create.UniqueConstraint("UQ").OnTable("TestTable").Column("Name").Clustered();
```

Note: You have to create the primary key index or unique constraint separately from the Create.Table expression to be able to specify them as clustered or non-clustered.

### Create a unique constraint on nullable columns using null value filter

Note: SQL Server has a different approach to creating unique constraints on nullable columns than ANSI standard.

```cs
Create.Index("UQ_NullFilter")
    .OnTable("TestTable").InSchema("dbo")
    .OnColumn("Name").Ascending()
    .WithOptions().Unique()
    .WithOptions().Filter("[Name] IS NOT NULL");
```

This will generate the following T-SQL DDL statement:

```sql
CREATE UNIQUE NONCLUSTERED INDEX [UQ_NullFilter] ON [dbo].[TestTable]
(
	[Name] ASC
)
WHERE ([Name] IS NOT NULL)
```

## Create a column of SQL data type `nvarchar(MAX)` / `varchar(MAX)`

Use `int.MaxValue` to represent infinite length strings.

```cs
Create.Column("Name").OnTable("TestTable").AsString(int.MaxValue);
```

Use `.AsAnsiString(int.MaxValue)` for `varchar(MAX)`:

```cs
Create.Column("Name").OnTable("TestTable").AsAnsiString(int.MaxValue);
```

Alternatively, you can define an extension method for use with `Create.Table` syntax, which would then be visible in the Visual Studio IntelliSense code completion dropdown:

```cs
public static class ICreateTableColumnAsTypeSyntaxExtensions
{
    public static ICreateTableColumnOptionOrWithColumnSyntax AsMaxString(this ICreateTableColumnAsTypeSyntax createTableColumnAsTypeSyntax)
    {
        return createTableColumnAsTypeSyntax.AsString(int.MaxValue);
    }
}
```

## WithIdentityInsert

If you want to turn on Identity Insert to be able to insert values into an identity column (see [here](http://msdn.microsoft.com/en-us/library/ms188059.aspx) for more details) then FluentMigrator has an extension method that supports this.

```cs
Insert.IntoTable("Foo")
  .WithIdentityInsert()
  .Row(new { id = 1, name = "Foo 1" });
```

## Add Identity Column with Seed and Increment

If you want to create an identity column and specify the seed (the id to start with) and an increment (how much the id value should increase when inserting new rows) then use this extension method:

```cs
Create.Table("TestTable")
  .WithColumn("Id").AsInt32().Identity(100,1)
```
