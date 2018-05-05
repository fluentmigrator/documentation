---
uid: tech-fluent-api-create
title: Fluent API (Create)
---

# [ICreateExpressionRoot](xref:FluentMigrator.Builders.Create.ICreateExpressionRoot)

<pre>
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
</pre>

- [ICreateSchemaOptionsSyntax](#icreateschemaoptionssyntax)
- [ICreateTableWithColumnOrSchemaOrDescriptionSyntax](#icreatetablewithcolumnorschemaordescriptionsyntax)
- [ICreateColumnOnTableSyntax](#icreatecolumnontablesyntax)
- [ICreateForeignKeyFromTableSyntax](#icreateforeignkeyfromtablesyntax)
- [ICreateIndexForTableSyntax](#icreateindexfortablesyntax)
- [ICreateSequenceInSchemaSyntax](#icreatesequenceinschemasyntax)
- [ICreateConstraintOnTableSyntax](#icreateconstraintontablesyntax)

# Schema

## ICreateSchemaOptionsSyntax

> [!NOTE]
> Extension point

# Table

## ICreateTableWithColumnOrSchemaOrDescriptionSyntax

<pre>
--------+-----------------------------+- ICreateTableWithColumnOrSchemaSyntax ---
        |                             |
        +- WithDescription ---- name -+
</pre>

## ICreateTableWithColumnOrSchemaSyntax

<pre>
--------+-----------------------------+- ICreateTableWithColumnSyntax ---
        |                             |
        +- InSchema ----------- name -+
</pre>

## ICreateTableWithColumnSyntax

<pre>
---------- WithColumn --------- name --- ICreateTableColumnAsTypeSyntax ---
</pre>

## ICreateTableColumnAsTypeSyntax

<pre>
---------- IColumnTypeSyntax ---- TNext: ICreateTableColumnOptionOrWithColumnSyntax ---
</pre>

- [IColumnTypeSyntax](xref:tech-fluent-api-common#icolumntypesyntaxtnext)

## ICreateTableColumnOptionOrWithColumnSyntax

<pre>
--------+- ICreateTableWithColumnSyntax -------------------------------------------------------------------+-
        |                                                                                                  |
        +- IColumnOptionSyntax -+- TNext: ICreateTableColumnOptionOrWithColumnSyntax ----------------------+
                                |                                                                          |
                                +- TNextFk: ICreateTableColumnOptionOrForeignKeyCascadeOrWithColumnSyntax -+
</pre>

- [IColumnOptionSyntax](xref:tech-fluent-api-common#icolumnoptionsyntaxtnexttnextfk)

## ICreateTableColumnOptionOrForeignKeyCascadeOrWithColumnSyntax

<pre>
--------+--> ICreateTableColumnOptionOrWithColumnSyntax
        |
        +- IForeignKeyCascadeSyntax -+--> TNext: ICreateTableColumnOptionOrWithColumnSyntax
                                     |
                                     +--> TNextFk: ICreateTableColumnOptionOrForeignKeyCascadeOrWithColumnSyntax
</pre>

- [IForeignKeyCascadeSyntax](xref:tech-fluent-api-common#iforeignkeycascadesyntaxtnexttnextfk)

# Column

## ICreateColumnOnTableSyntax

<pre>
---------- OnTable --- name ---> ICreateColumnAsTypeOrInSchemaSyntax
</pre>

## ICreateColumnAsTypeOrInSchemaSyntax

<pre>
--------+---------------------+--> ICreateColumnAsTypeSyntax
        |                     |
        +- InSchema --- name -+
</pre>

## ICreateColumnAsTypeSyntax

<pre>
---------- IColumnTypeSyntax ---> ICreateColumnOptionSyntax
</pre>

- [IColumnTypeSyntax](xref:tech-fluent-api-common#icolumntypesyntaxtnext)
- [ICreateColumnOptionSyntax](#icreatecolumnoptionsyntax)

## ICreateColumnOptionSyntax

<pre>
--------+-------------------------------+- IColumnOptionSyntax -+--> TNext: ICreateColumnOptionSyntax
        |                               |                       |
        +- SetExistingRowsTo --- value -+                       +--> TNextFk: ICreateColumnOptionOrForeignKeyCascadeSyntax
</pre>

- [IColumnOptionSyntax](xref:tech-fluent-api-common#icolumnoptionsyntaxtnexttnextfk)

## ICreateColumnOptionOrForeignKeyCascadeSyntax

<pre>
--------+--> ICreateColumnOptionSyntax
        |
        +- IForeignKeyCascadeSyntax -+--> TNext: ICreateColumnOptionSyntax
                                     |
                                     +--> TNextFk: ICreateColumnOptionOrForeignKeyCascadeSyntax
</pre>

- [IForeignKeyCascadeSyntax](xref:tech-fluent-api-common#iforeignkeycascadesyntaxtnexttnextfk)

# ForeignKey

## ICreateForeignKeyFromTableSyntax

<pre>
---------- FromTable --- table ---> ICreateForeignKeyForeignColumnOrInSchemaSyntax
</pre>

## ICreateForeignKeyForeignColumnOrInSchemaSyntax

<pre>
--------+---------------------+--> ICreateForeignKeyForeignColumnSyntax
        |                     |
        +- InSchema --- name -+
</pre>

## ICreateForeignKeyForeignColumnSyntax

<pre>
--------+- ForeignColumn --- column ---+--> ICreateForeignKeyToTableSyntax
        |                              |
        +- ForeignColumns --- columns -+
</pre>

## ICreateForeignKeyToTableSyntax

<pre>
---------- ToTable name --> ICreateForeignKeyPrimaryColumnOrInSchemaSyntax
</pre>

## ICreateForeignKeyPrimaryColumnOrInSchemaSyntax

<pre>
--------+-----------------+--> ICreateForeignKeyPrimaryColumnSyntax
        |                 |
        +- InSchema name -+
</pre>

## ICreateForeignKeyPrimaryColumnSyntax

<pre>
--------+- PrimaryColumn --- column ---+--> ICreateForeignKeyCascadeSyntax
        |                              |
        +- PrimaryColumns --- columns -+
</pre>

## ICreateForeignKeyCascadeSyntax

<pre>
--------+- OnDeleteOrUpdate --- rule ----| (end)
        |
        +- OnDelete --- rule ---------+--> ICreateForeignKeyCascadeSyntax
        |                             |
        +- OnUpdate --- rule ---------+
</pre>

# Index

## ICreateIndexForTableSyntax

<pre>
---------- OnTable --- name ---> ICreateIndexOnColumnOrInSchemaSyntax
</pre>

## ICreateIndexOnColumnOrInSchemaSyntax

<pre>
--------+---------------------+--> ICreateIndexOnColumnSyntax
        |                     |
        +- InSchema --- name -+
</pre>

## ICreateIndexOnColumnSyntax

<pre>
--------+- OnColumn --- name ---> ICreateIndexColumnOptionsSyntax
        |
        +- WithOptions ---------> ICreateIndexOptionsSyntax
</pre>

## ICreateIndexColumnOptionsSyntax

<pre>
--------+- Ascending --+--> ICreateIndexMoreColumnOptionsSyntax
        |              |
        +- Descending -+
        |
        +- Unique --------> ICreateIndexColumnUniqueOptionsSyntax
</pre>

## ICreateIndexOptionsSyntax

<pre>
--------+- Unique -------+--> ICreateIndexOnColumnSyntax
        |                |
        +- NonClustered -+
        |                |
        +- Clustered ----+
</pre>

- [ICreateIndexOnColumnSyntax](#icreateindexoncolumnsyntax)

## ICreateIndexMoreColumnOptionsSyntax

> [!NOTE]
> Extension point

<pre>
--------+---------------------+--> ICreateIndexOnColumnSyntax
        |                     |
        +- get_CurrentColumn -+
</pre>

- [ICreateIndexOnColumnSyntax](#icreateindexoncolumnsyntax)

## ICreateIndexColumnUniqueOptionsSyntax

> [!NOTE]
> Extension point

<pre>
--------+---------------------+--> ICreateIndexOnColumnSyntax
        |                     |
        +- get_CurrentColumn -+
</pre>

- [ICreateIndexOnColumnSyntax](#icreateindexoncolumnsyntax)

# Sequence

## ICreateSequenceInSchemaSyntax

<pre>
--------+---------------------+--> ICreateSequenceSyntax
        |                     |
        +- InSchema --- name -+
</pre>

## ICreateSequenceSyntax

<pre>
--------+-----------------------------+--> ICreateSequenceSyntax
        |                             |
        +- IncrementBy --- increment -+
        |                             |
        +- MinValue --- minValue -----+
        |                             |
        +- MaxValue --- maxValue -----+
        |                             |
        +- StartWith --- startWith ---+
        |                             |
        +- Cache --- value -----------+
        |                             |
        +- Cycle ---------------------+
</pre>

# PrimaryKey/Unique Constraint

## ICreateConstraintOnTableSyntax

<pre>
---------- OnTable --- name ----> ICreateConstraintWithSchemaOrColumnSyntax
</pre>

## ICreateConstraintWithSchemaOrColumnSyntax

<pre>
--------+- ICreateConstraintColumnsSyntax
        |
        +- ICreateConstraintWithSchemaSyntax
</pre>

Expanded:

<pre>
--------+-----------------------+--+- Column --- name ---+--> ICreateConstraintOptionsSyntax
        |                       |  |                     |
        +- WithSchema --- name -+  +- Columns --- names -+
</pre>

## ICreateConstraintColumnsSyntax

<pre>
--------+- Column --- name ---+--> ICreateConstraintOptionsSyntax
        |                     |
        +- Columns --- names -+
</pre>

## ICreateConstraintWithSchemaSyntax

<pre>
---------- WithSchema --- name ----> ICreateConstraintColumnsSyntax
</pre>

## ICreateConstraintOptionsSyntax

> [!NOTE]
> Extension point
