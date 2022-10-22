---
uid: faq-change-naming-conventions.md
title: "How To: Change Naming Conventions"
---

FluentMigrator supports changing naming conventions for:
1. Columns
2. Constraints
3. Indexes
4. Sequences
5. Schemas
6. Auto Names
7. Root Paths

While FluentMigrator comes with reasonable naming conventions, some organizations may have strict standards.
In such cases, overriding FluentMigrator's opinion may be desirable.

# High-Level Tasks

1. Implement the `IConventionSet` interface (example: see below)
2. Register the `IConventionSet` as singleton: `services.AddSingleton<IConventionSet>(new YourConventionSet())`

# Example `IConventionSet` implementation
```cs
    public class YourConventionSet : IConventionSet
    {
        public YourConventionSet()
            : this(new DefaultConventionSet())
        {
        }

        public YourConventionSet(IConventionSet innerConventionSet)
        {
            ForeignKeyConventions = new List<IForeignKeyConvention>()
            {
                /* This is where you do your stuff */
                new YourCustomDefaultForeignKeyNameConvention(),
                innerConventionSet.SchemaConvention,
            };

            ColumnsConventions = innerConventionSet.ColumnsConventions;
            ConstraintConventions = innerConventionSet.ConstraintConventions;
            IndexConventions = innerConventionSet.IndexConventions;
            SequenceConventions = innerConventionSet.SequenceConventions;
            AutoNameConventions = innerConventionSet.AutoNameConventions;
            SchemaConvention = innerConventionSet.SchemaConvention;
            RootPathConvention = innerConventionSet.RootPathConvention;
        }

        /// <inheritdoc />
        public IRootPathConvention RootPathConvention { get; }

        /// <inheritdoc />
        public DefaultSchemaConvention SchemaConvention { get; }

        /// <inheritdoc />
        public IList<IColumnsConvention> ColumnsConventions { get; }

        /// <inheritdoc />
        public IList<IConstraintConvention> ConstraintConventions { get; }

        /// <inheritdoc />
        public IList<IForeignKeyConvention> ForeignKeyConventions { get; }

        /// <inheritdoc />
        public IList<IIndexConvention> IndexConventions { get; }

        /// <inheritdoc />
        public IList<ISequenceConvention> SequenceConventions { get; }

        /// <inheritdoc />
        public IList<IAutoNameConvention> AutoNameConventions { get; }
    }
```
