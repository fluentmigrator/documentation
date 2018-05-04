---
uid: tech-fluent-api
title: Fluent API
---

# How does the fluent API work?

The starting points for the fluent API are the following classes:

- [MigrationBase](xref:FluentMigrator.MigrationBase): database modeling
- [Migration](xref:FluentMigrator.Migration): data modification and script execution

# Supported expressions

## Create ([ICreateExpressionRoot](xref:FluentMigrator.Builders.Create.ICreateExpressionRoot))

Create a database object.

### [ICreateExpressionRoot](xref:FluentMigrator.Builders.Create.ICreateExpressionRoot)

```
Create -+- Schema ------------- name --- ICreateSchemaOptionsSyntax ---
        |
        +- Table -------------- name --- ICreateTableWithColumnOrSchemaOrDescriptionSyntax ---
        |
        +- Column ------------- name --- ICreateColumnOnTableSyntax ---
        |
        +- ForeignKey -------+--------+- ICreateForeignKeyFromTableSyntax ---
        |                    |        |
        |                    +- name -+
        |
        +- Index ------------+--------+- ICreateIndexForTableSyntax ---
        |                    |        |
        |                    +- name -+
        |
        +- Sequence ----------- name --- ICreateSequenceInSchemaSyntax ---
        |
        +- PrimaryKey -------+--------+- ICreateConstraintOnTableSyntax ---
        |                    |        |
        |                    +- name -+
        |
        +- UniqueConstraint -+--------+- ICreateConstraintOnTableSyntax ---
                             |        |
                             +- name -+
```

### ICreateSchemaOptionsSyntax

This is just an extension point.

### ICreateTableWithColumnOrSchemaOrDescriptionSyntax

```
--------+-----------------------------+- ICreateTableWithColumnOrSchemaSyntax ---
        |                             |
        +- WithDescription ---- name -+
```

### ICreateTableWithColumnOrSchemaSyntax

```
--------+-----------------------------+- ICreateTableWithColumnSyntax ---
        |                             |
        +- InSchema ----------- name -+
```

### ICreateTableWithColumnSyntax

```
---------- WithColumn --------- name --- ICreateTableColumnAsTypeSyntax ---
```

### ICreateTableColumnAsTypeSyntax

```
---------- IColumnTypeSyntax ---- TNext: ICreateTableColumnOptionOrWithColumnSyntax ---
```

### ICreateColumnOptionSyntax

```
--------+-------------------------------+- IColumnOptionSyntax -+--> TNext: ICreateColumnOptionSyntax
        |                               |                       |
        +- SetExistingRowsTo --- value -+                       +--> TNextFk: ICreateColumnOptionOrForeignKeyCascadeSyntax
```

### ICreateColumnOptionOrForeignKeyCascadeSyntax

```
--------+--> ICreateColumnOptionSyntax
        |
        +- IForeignKeyCascadeSyntax -+--> TNext: ICreateColumnOptionSyntax
                                     |
                                     +--> TNextFk: ICreateColumnOptionOrForeignKeyCascadeSyntax
```

### ICreateTableColumnOptionOrWithColumnSyntax

```
--------+- ICreateTableWithColumnSyntax -------------------------------------------------------------------+-
        |                                                                                                  |
        +- IColumnOptionSyntax -+- TNext: ICreateTableColumnOptionOrWithColumnSyntax ----------------------+
                                |                                                                          |
                                +- TNextFk: ICreateTableColumnOptionOrForeignKeyCascadeOrWithColumnSyntax -+
```

### ICreateTableColumnOptionOrForeignKeyCascadeOrWithColumnSyntax

```
--------+--> ICreateTableColumnOptionOrWithColumnSyntax
        |
        +- IForeignKeyCascadeSyntax -+--> TNext: ICreateTableColumnOptionOrWithColumnSyntax
                                     |
                                     +--> TNextFk: ICreateTableColumnOptionOrForeignKeyCascadeOrWithColumnSyntax
```

### ICreateColumnOnTableSyntax

```
---------- OnTable --- name ---> ICreateColumnAsTypeOrInSchemaSyntax
```

### ICreateColumnAsTypeOrInSchemaSyntax

```
--------+---------------------+--> ICreateColumnAsTypeSyntax
        |                     |
        +- InSchema --- name -+
```

### ICreateColumnAsTypeSyntax

```
---------- IColumnTypeSyntax ---> ICreateColumnOptionSyntax
```

### ICreateForeignKeyFromTableSyntax

```
```

## Alter ([IAlterExpressionRoot](xref:FluentMigrator.Builders.Alter.IAlterExpressionRoot))

Alter a database object.

## Delete ([IDeleteExpressionRoot](xref:FluentMigrator.Builders.Delete.IDeleteExpressionRoot))

Delete a database object or data.

## Rename ([IRenameExpressionRoot](xref:FluentMigrator.Builders.Rename.IRenameExpressionRoot))

Rename a database object.

## Schema ([ISchemaExpressionRoot](xref:FluentMigrator.Builders.Schema.ISchemaExpressionRoot))

Schema-oriented starting point for expressions.

## IfDatabase ([IIfDatabaseExpressionRoot](xref:FluentMigrator.Builders.IfDatabase.IIfDatabaseExpressionRoot))

Create database-specific expressions.

## Insert ([IInsertExpressionRoot](xref:FluentMigrator.Builders.Insert.IInsertExpressionRoot))

Insert data.

## Update ([IUpdateExpressionRoot](xref:FluentMigrator.Builders.Update.IUpdateExpressionRoot))

Update data.

## Execute ([IExecuteExpressionRoot](xref:FluentMigrator.Builders.Execute.IExecuteExpressionRoot))

Execute SQL statements/scripts.

# Generic interfaces

## IColumnTypeSyntax&lt;TNext&gt;

```
--------+- AsAnsiString -+--------+-------------+---------------+-> TNext
        |                |        |             |               |
        |                +- size -+- collation -+               |
        |                                                       |
        +- AsBinary -----+--------+-----------------------------+
        |                |        |                             |
        |                +- size -+                             |
        |                                                       |
        +- AsBoolean -------------------------------------------+
        |                                                       |
        +- AsByte ----------------------------------------------+
        |                                                       |
        +- AsCurrency ------------------------------------------+
        |                                                       |
        +- AsDate ----------------------------------------------+
        |                                                       |
        +- AsDateTime ------------------------------------------+
        |                                                       |
        +- AsDateTime2 -----------------------------------------+
        |                                                       |
        +- AsDateTimeOffset ------------------------------------+
        |                                                       |
        +- AsDecimal -------------------------------------------+
        |                                                       |
        +- AsDouble --------------------------------------------+
        |                                                       |
        +- AsGuid ----------------------------------------------+
        |                                                       |
        +- AsFixedLengthString --- size -----+-------------+----+
        |                                    |             |    |
        |                                    +- collation -+    |
        |                                                       |
        +- AsFixedLengthAnsiString --- size -+-------------+----+
        |                                    |             |    |
        |                                    +- collation -+    |
        |                                                       |
        +- AsFloat ---------------------------------------------+
        |                                                       |
        +- AsInt16 ---------------------------------------------+
        |                                                       |
        +- AsInt32 ---------------------------------------------+
        |                                                       |
        +- AsInt64 ---------------------------------------------+
        |                                                       |
        +- AsString -----+--------+-------------+---------------+
        |                |        |             |               |
        |                +- size -+- collation -+               |
        |                                                       |
        +- AsTime ----------------------------------------------+
        |                                                       |
        +- AsXml --------+--------+-----------------------------+
        |                |        |                             |
        |                +- size -+                             |
        |                                                       |
        +- AsCustom --- customType -----------------------------+
```

## IColumnOptionSyntax&lt;TNext,TNextFk&gt;

```
--------+- WithDefault --- method --------------------------------------+--> TNext
        |                                                               |
        +- WithDefaultValue --- value ----------------------------------+
        |                                                               |
        +- WithColumnDescription --- description -----------------------+
        |                                                               |
        +- Identity ----------------------------------------------------+
        |                                                               |
        +- Indexed -----------------------------------------------------+
        |                                                               |
        +- PrimaryKey -+--------+---------------------------------------+
        |              |        |                                       |
        |              +- name -+                                       |
        |                                                               |
        +- Nullable ----------------------------------------------------+
        |                                                               |
        +- NotNullable -------------------------------------------------+
        |                                                               |
        +- Unique -----+--------+---------------------------------------+
        |              |        |
        |              +- name -+
        |
        +- ForeignKey ---+-------------------------------------------+--+--> TNextFk
        |                |                                           |  |
        |                +-----------------------+- table --- column-+  |
        |                |                       |                      |
        |                +- name -+-+----------+-+                      |
        |                           |          |                        |
        |                           +- schema -+                        |
        |                                                               |
        +- ReferencedBy -+-----------------------+- table --- column-+--+
                         |                       |
                         +- name -+-+----------+-+
                                    |          |
                                    +- schema -+
```

## IForeignKeyCascadeSyntax&lt;TNext,TNextFk&gt;

```
--------+- OnDeleteOrUpdate --- rule ---> TNext
        |
        +- OnDelete --- rule -+---------> TNextFk
        |                     |
        +- OnUpdate --- rule -+
```

# Planned changes

- Differentiate between `Delete` and `Drop`

