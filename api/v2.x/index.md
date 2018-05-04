# API for version 2.x

# Using the in-process runner

```cs
// Create the announcer to output the migration messages
var announcer = new ConsoleAnnouncer();

// Processor specific options (usually none are needed)
var options = new ProcessorOptions();

// Use SQLite
var processorFactory = new SQLiteProcessorFactory();

// Initialize the processor
using (var processor = processorFactory.Create(
    // The SQLite connection string
    "Data Source=test.db",
    announcer,
    options))
{
    // Configure the runner
    var context = new RunnerContext(announcer);

    // Create the migration runner
    var runner = new MigrationRunner(
        // Specify the assembly with the migrations
        typeof(MyMigration).Assembly,
        context,
        processor);

    // Run the migrations
    runner.MigrateUp();
}
```
