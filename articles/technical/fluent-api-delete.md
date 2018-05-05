---
uid: tech-fluent-api-delete
title: Fluent API (Delete)
---

# [IDeleteExpressionRoot](xref:FluentMigrator.Builders.Delete.IDeleteExpressionRoot)

<pre>
Delete -+- Schema ------------- name ----------| (end)
        |
        +- Table -------------- name -------+--> IInSchemaSyntax
        |                                   |
        +- Sequence ----------- name -------+
        |
        +- Column ------------- name ----------> IDeleteColumnFromTableSyntax
        |
        +- ForeignKey -------+-----------------> IDeleteForeignKeyFromTableSyntax
        |                    |
        |                    +- name ----------> IDeleteForeignKeyOnTableSyntax
        |
        +- FromTable ---------- name ----------> IDeleteDataOrInSchemaSyntax
        |
        +- Index ------------+--------+--------> IDeleteIndexForTableSyntax
        |                    |        |
        |                    +- name -+
        |
        +- PrimaryKey --------- name -------+--> IDeleteConstraintOnTableSyntax
        |                                   |
        +- UniqueConstraint -+--------+-----+
        |                    |        |
        |                    +- name -+
        |
        +- DefaultConstraint ------------------> IDeleteDefaultConstraintOnTableSyntax
</pre>

# Generic

## IInSchemaSyntax

<pre>
---------- InSchema --- name ---| (end)
</pre>

# Column

## IDeleteColumnFromTableSyntax

<pre>
--------+- FromTable --- name ---> IInSchemaSyntax
        |
        +- Column --- name ------> IDeleteColumnFromTableSyntax
</pre>

# ForeignKey

## IDeleteForeignKeyFromTableSyntax

<pre>
---------- FromTable --- name ---> IDeleteForeignKeyForeignColumnOrInSchemaSyntax
</pre>

## IDeleteForeignKeyForeignColumnOrInSchemaSyntax

<pre>
--------+-------------------+--> IDeleteForeignKeyForeignColumnSyntax
        |                   |
        +- InSchema name ---+
</pre>

## IDeleteForeignKeyForeignColumnSyntax

<pre>
--------+- ForeignColumn ---- name --+--> IDeleteForeignKeyToTableSyntax
        |                            |
        +- ForeignColumns --- names -+
</pre>

## IDeleteForeignKeyToTableSyntax

<pre>
---------- ToTable --- name ---> IDeleteForeignKeyPrimaryColumnSyntax
</pre>

## IDeleteForeignKeyPrimaryColumnSyntax

<pre>
--------+- PrimaryColumn ---- name --+--| (end)
        |                            |
        +- PrimaryColumns --- names -+
</pre>

## IDeleteForeignKeyOnTableSyntax

<pre>
---------- OnTable --- name ---> IInSchemaSyntax
</pre>

# Data

## IDeleteDataOrInSchemaSyntax

<pre>
--------+---------------------+--> IDeleteDataSyntax
        |                     |
        +- InSchema --- name -+
</pre>

## IDeleteDataSyntax

<pre>
--------+- Row --- anonymousType ----> IDeleteDataSyntax
        |
        +- AllRows ---------------+--| (end)
        |                         |
        +- IsNull --- columnName -+
</pre>

# Index

## IDeleteIndexForTableSyntax

<pre>
--------+- OnTable --- name ---> IDeleteIndexOnColumnOrInSchemaSyntax
        |
        +- WithOptions --------> IDeleteIndexOptionsSyntax
</pre>

## IDeleteIndexOnColumnOrInSchemaSyntax

<pre>
--------+---------------------+--> IDeleteIndexOnColumnSyntax
        |                     |
        +- InSchema --- name -+
</pre>

## IDeleteIndexOnColumnSyntax

<pre>
--------+- OnColumn --- name ---+--> IDeleteIndexOptionsSyntax
        |                       |
        +- OnColumns --- names -+
        |                       |
        +- WithOptions ---------+
</pre>

## IDeleteIndexOptionsSyntax

> [!NOTE]
> Extension point

# PrimaryKey/Unique constraint

## IDeleteConstraintOnTableSyntax

<pre>
---------- FromTable name ---> IDeleteConstraintInSchemaOptionsSyntax
</pre>

## IDeleteConstraintInSchemaOptionsSyntax

<pre>
--------+---------------------+--> IDeleteConstraintColumnSyntax
        |                     |
        +- InSchema --- name -+
</pre>

## IDeleteConstraintColumnSyntax

<pre>
--------+- OnColumn --- name ---+--| (end)
        |                       |
        +- OnColumns --- names -+
</pre>

# DefaultConstraint

## IDeleteDefaultConstraintOnTableSyntax

<pre>
---------- OnTable --- name ---> IDeleteDefaultConstraintOnColumnOrInSchemaSyntax
</pre>

## IDeleteDefaultConstraintOnColumnOrInSchemaSyntax

<pre>
--------+---------------------+--> IDeleteDefaultConstraintOnColumnSyntax
        |                     |
        +- InSchema --- name -+
</pre>

## IDeleteDefaultConstraintOnColumnSyntax

<pre>
---------- OnColumn --- name ---| (end)
</pre>
