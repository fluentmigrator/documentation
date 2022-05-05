---
uid: comparison-to-entity-framework-migrations
title: FluentMigrator Comparison to Entity Framework Code-First Migrations
---


Technically, FluentMigrator does not "generate" migrations for you. You have to write them yourself. By comparison, Entity Framework Core can use its fluent model building syntax and automatically infer up/down migrations, and you can focus primarily on writing data migrations.  Entity Framework Core does this through PowerShell command-line interface `Add-Migration`, which keeps track of a ModelSnapshot state.

That said, here is how Microsoft recommends writing migrations with EFCore, compared with the same logic in FluentMigrator:

# EFCore
Source: https://docs.microsoft.com/en-us/aspnet/core/data/ef-mvc/migrations?view=aspnetcore-3.1#examine-up-and-down-methods
```c#
public partial class InitialCreate : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.CreateTable(
            name: "Course",
            columns: table => new
            {
                CourseID = table.Column<int>(nullable: false),
                Credits = table.Column<int>(nullable: false),
                Title = table.Column<string>(nullable: true)
            },
            constraints: table =>
            {
                table.PrimaryKey("PK_Course", x => x.CourseID);
            });

        // Additional code not shown
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.DropTable(
            name: "Enrollment");
        // Additional code not shown
    }
}
```

# FluentMigrator

```c#
[Migration(1, "InitialCreate")]
public class InitialCreate : Migration
{
    public void Up()
    {
        Create.Table("Course")
                .WithColumn("CourseID").AsInt32().NotNullable().PrimaryKey()
                .WithColumn("Credits").AsInt32().NotNullable()
                .WithColumn("Title").AsString().Nullable();

        // Additional code not shown
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        Delete.Table("Enrollment");
        // Additional code not shown
    }
}
```

In the above example, you don't need to type out `PK_Course` in your primary key definition, because FluentMigrator supports "Don't Repeat Yourself" principle via naming conventions and "knows" that is the correct name for your primary key.  You just define your naming conventions once.

Also, because of the fluent syntax, you'll find you have to type a lot less than you do with EFCore migrations.  If you find this to not be the case, please provide feedback and we will make it more efficient.
