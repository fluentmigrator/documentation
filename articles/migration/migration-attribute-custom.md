---
uid: migration-attribute-custom
title: Enforcing migration version numbering rules
---

# Enforcing migration version numbering rules

Frequently, it is useful for a project to employ a specific system for migration version numbering. A useful technique for simplifying and encoding this system is with a custom attribute that inherits from [MigrationAttribute](xref:FluentMigrator.MigrationAttribute).  

An example is below.  This uses a system where the migration version is of the format BBYYYYMMDDHHMM, where BB is a branch, YYYY is year, MM is month, and so on.


```csharp
/// <summary>
/// Mark all migrations with this INSTEAD of [Migration].
/// </summary>
public class MyCustomMigrationAttribute : FluentMigrator.MigrationAttribute
{
    public MyCustomMigrationAttribute(int branchNumber, int year, int month, int day, int hour, int minute, string author)
       : base(CalculateValue(branchNumber, year, month, day, hour, minute))
   {
       this.Author = author;
   }
   public string Author { get; private set; }
   private static long CalculateValue(int branchNumber, int year, int month, int day, int hour, int minute)
   {
      return branchNumber * 1000000000000L + year * 100000000L + month * 1000000L + day * 10000L + hour * 100L + minute;
   }
}
```

A migration class that uses this attribute might look like this:

```csharp
[MyCustomMigration(author: "Scott Stafford", branchNumber: 12, year: 2012, month: 8, day: 7, hour: 14, minute: 01)]
public class TestLcmpMigration : Migration
{
    public override void Down() { /* ... */ }
    public override void Up() { /* ... */ }
}
```

Note, this example also contains an additional attribute property, Author, to demonstrate that you can have additional properties. In practice, adding an Author may not be entirely useful, since it will not be automatically added to the VersionInfo table as part of the audit log of what was executed against the target database.  However, if developers were to tag the migrations they wrote with the Author field, it might assist in troubleshooting production issues. As a best practice, these issues should be caught by a good CI/CD process that involves continuously building the initial full database schema to create a baseline, and then applying all migrations. In this way, hopefully nobody is going to wake "Scott Stafford" up in the middle of the night due to a production issue :)