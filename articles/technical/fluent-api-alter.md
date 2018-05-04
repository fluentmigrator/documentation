---
uid: tech-fluent-api-alter
title: Fluent API (Alter)
---

# Alter ([IAlterExpressionRoot](xref:FluentMigrator.Builders.Alter.IAlterExpressionRoot))

<pre>
Alter --+- Table --- name ----> IAlterTableAddColumnOrAlterColumnOrSchemaOrDescriptionSyntax
        |
        +- Column --- name ---> IAlterColumnOnTableSyntax
</pre>

## IAlterTableAddColumnOrAlterColumnOrSchemaOrDescriptionSyntax

<pre>
--------+-----------------------------------+--> IAlterTableAddColumnOrAlterColumnOrSchemaSyntax
        |                                   |
        +- WithDescription --- description -+
</pre>

## IAlterColumnOnTableSyntax

<pre>
---------- OnTable --- name --> IAlterColumnAsTypeOrInSchemaSyntax
</pre>

## IAlterColumnAsTypeOrInSchemaSyntax

<pre>
--------+-----------------------+--> IAlterColumnAsTypeSyntax
        |                       |
        +- InSchema --- schema -+
</pre>

## IAlterColumnAsTypeSyntax

<pre>
---------- IColumnTypeSyntax ---- TNext: IAlterColumnOptionSyntax ---
</pre>

- [IColumnTypeSyntax](xref:tech-fluent-api-common#icolumntypesyntaxtnext)

## IAlterColumnOptionSyntax

<pre>
---------- IColumnOptionSyntax -+--> TNext: IAlterColumnOptionSyntax
                                |
                                +--> TNextFk: IAlterColumnOptionOrForeignKeyCascadeSyntax
</pre>

- [IColumnOptionSyntax](xref:tech-fluent-api-common#icolumnoptionsyntaxtnexttnextfk)

## IAlterColumnOptionOrForeignKeyCascadeSyntax

<pre>
--------+--> IAlterColumnOptionSyntax
        |
        +- IForeignKeyCascadeSyntax -+--> TNext: IAlterColumnOptionSyntax
                                     |
                                     +--> TNextFk: IAlterColumnOptionOrForeignKeyCascadeSyntax
</pre>

- [IForeignKeyCascadeSyntax](xref:tech-fluent-api-common#iforeignkeycascadesyntaxtnexttnextfk)

## IAlterTableAddColumnOrAlterColumnOrSchemaSyntax

<pre>
--------+-----------------------+--> IAlterTableAddColumnOrAlterColumnSyntax
        |                       |
        +- InSchema --- schema -+
</pre>

## IAlterTableAddColumnOrAlterColumnSyntax

<pre>
--------+- AddColumn name ---+--> IAlterTableColumnAsTypeSyntax
        |                    |
        +- AlterColumn name -+
        |
        +- ToSchema name -------| (end)
</pre>

## IAlterTableColumnAsTypeSyntax

<pre>
---------- IColumnTypeSyntax ---- TNext: IAlterTableColumnOptionOrAddColumnOrAlterColumnSyntax ---
</pre>

- [IColumnTypeSyntax](xref:tech-fluent-api-common#icolumntypesyntaxtnext)

## IAlterTableColumnOptionOrAddColumnOrAlterColumnSyntax

<pre>
--------+-------------------------------+--+--> IAlterTableAddColumnOrAlterColumnSyntax
        |                               |  |
        +- SetExistingRowsTo --- value -+  +--> IColumnOptionSyntax -+--> TNext: IAlterTableColumnOptionOrAddColumnOrAlterColumnSyntax
                                                                     |
                                                                     +--> TNextFk: IAlterTableColumnOptionOrAddColumnOrAlterColumnOrForeignKeyCascadeSyntax
</pre>

- [IColumnOptionSyntax](xref:tech-fluent-api-common#icolumnoptionsyntaxtnexttnextfk)

## IAlterTableColumnOptionOrAddColumnOrAlterColumnOrForeignKeyCascadeSyntax

<pre>
--------+--> IAlterTableColumnOptionOrAddColumnOrAlterColumnSyntax
        |
        +- IForeignKeyCascadeSyntax -+--> TNext: IAlterTableColumnOptionOrAddColumnOrAlterColumnSyntax
                                     |
                                     +--> TNextFk: IAlterTableColumnOptionOrAddColumnOrAlterColumnOrForeignKeyCascadeSyntax
</pre>

- [IForeignKeyCascadeSyntax](xref:tech-fluent-api-common#iforeignkeycascadesyntaxtnexttnextfk)
