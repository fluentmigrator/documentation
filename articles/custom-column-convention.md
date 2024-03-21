---
uid: custom-column-convention
title: Creating a Custom ColumnConvention
---

# Creating a Custom Column Convention

In certain instances, it may be required to add your own customized column conventions. This is
useful for supporting custom column formats that aren't covered by FluentMigrator

First, create your custom column convention by implementing `IColumnsConvention`. In this example, a column convention is added that supoprts a JSON column like so:

```csharp
class MyCustomColumnConvention : IColumnsConvention {
  private readonly string _databaseType;

  public MyCustomColumnConvention(IProcessorAccessor processorAccessor) {
    _databaseType = processorAccessor.Processor.DatabaseType.ToLowerInvariant();
  }

  public IColumnsExpression Apply(IColumnsExpression expression) {
    var jsonColumns = expression.Columns
      .Where(x => string.Equals("json", x.CustomType, StringComparison.OrdinalIgnoreCase))
      .ToList();
    if (jsonColumns.Count == 0) {
      return expression;
    }

    if (_databaseType == "sqlserver2016") {
      foreach (var column in jsonColumns) {
        column.CustomType = null;
        column.Type = DbType.String;
        column.Size = int.MaxValue;
      }
    }

    return expression;
  }
}
```

Option 1, useful if you already depend on behavior of `IConventionSetAccessor`, Add your convention to the existing IConventionSet:

```csharp
var conventionSet = services.GetRequiredService<IConventionSet>();
var processorAccessor = services.GetRequiredService<IProcessorAccessor>();
conventionSet.ColumnConventions.Add(new MyCustomColumnConvention(processorAccessor));
```

Option 2, doesn't rely on assembly scanning and requires:

1. A custom `IConventionSet` as scoped service, which should be derived from `DefaultConventionSet` and have only one public constructor,
2. The custom `IColumnsConvention` must be added to the `ColumnsConventions` property of `IConventionSet`
