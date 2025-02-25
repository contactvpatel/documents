# **Database Migration with DbUp in .NET**
Here is a comprehensive end-to-end guide on using **DbUp** in a .NET API project for schema migration, including a rollback strategy and multiple iteration examples.

## Introduction
DbUp is a database migration tool that allows developers to manage schema changes in a controlled and automated manner. This document provides a detailed guide on how to use DbUp in a .NET API project for managing PostgreSQL database migrations, including its advantages and disadvantages.

For more information, visit the official DbUp documentation: [DbUp Documentation](https://dbup.readthedocs.io/en/latest/)

## Implementation Details

### 1. Installing DbUp NuGet Package
To use DbUp for PostgreSQL or SQL Server, install the appropriate NuGet package:

For PostgreSQL:
```bash
dotnet add package DbUp.Postgresql
```

For SQL Server:
```bash
dotnet add package DbUp.SqlServer
```

### 2. Setting Up DbUp Migration
**Note:** All migration, rollback, and seed data scripts must be idempotent to ensure safe re-execution without causing conflicts or duplication.
In the provided code, the `DatabaseMigrationInitFilter` class implements `IStartupFilter`, ensuring that database migrations are executed during application startup.

#### **Key Components**:
1. **Checking Migration Status**:
   - The migration process begins if `EnableDatabaseMigration` is set to `true` in the application settings.
   - Migration scripts are loaded from the `DbUpMigration/Migrations` directory.

2. **Ensuring Database Existence**:
   - `EnsureDatabase.For.PostgresqlDatabase(dbConnectionString);` ensures that the PostgreSQL database exists before applying migrations.

3. **Advisory Lock Mechanism**:
   - Implements `pg_try_advisory_lock()` to ensure that only one instance applies migrations in a concurrent environment.
   - The lock is held for up to 60 seconds while retrying every 5 seconds.

4. **Executing Migrations**:
   - Uses `DeployChanges.To.PostgresqlDatabase(dbConnectionString)` to apply scripts.
   - Runs migration scripts only if required.
   - Logs migration progress using `DbUpLoggerExtension`.

5. **Seeding Data**:
   - Seed scripts are executed from the `DbUpMigration/Seeds/<Environment>` directory.
   - Seed execution follows a similar approach to migrations.

6. **Transaction Support**:
   - Migrations and seeding are executed within transactions to maintain atomicity.

```csharp

using DbUp;
using Microsoft.Extensions.Options;
using Npgsql;
using PBR.Core.Models;
using PBR.Util.Logging;
using PBR.Util.Models;

namespace PBR.Api.Filters
{
    public class DatabaseMigrationInitFilter : IStartupFilter
    {
        private readonly IOptionsMonitor<AppSettingModel> _appSettingModel;
        private readonly IOptionsMonitor<DbConnectionModel> _dbConnectionModel;
        public readonly ILogger _logger;

        public DatabaseMigrationInitFilter(IOptionsMonitor<AppSettingModel> appSettingModel,
            IOptionsMonitor<DbConnectionModel> dbConnectionModel,
            ILogger<DatabaseMigrationInitFilter> logger)
        {
            _appSettingModel = appSettingModel ?? throw new ArgumentNullException(nameof(appSettingModel));
            _dbConnectionModel = dbConnectionModel ?? throw new ArgumentNullException(nameof(dbConnectionModel));
            _logger = logger ?? throw new ArgumentNullException(nameof(logger));
        }

        public Action<IApplicationBuilder> Configure(Action<IApplicationBuilder> next)
        {
            if (_appSettingModel.CurrentValue.EnableDatabaseMigration)
            {
                _logger.LogInformationExtension("Database migration is enabled; starting the migration process.");

                var dbConnectionString = DbConnectionModel.CreateConnectionString(_dbConnectionModel.CurrentValue.Read);

                var dbUpLogger = new DbUpLoggerExtension(_logger);

                string migrationPath = Path.Combine(AppDomain.CurrentDomain.BaseDirectory, "DbUpMigration", "Migrations");

                if (Directory.Exists(migrationPath) && Directory.EnumerateFiles(migrationPath).Any())
                {
                    // Ensure Database exists, if not then create it
                    EnsureDatabase.For.PostgresqlDatabase(dbConnectionString);

                    using var connection = new NpgsqlConnection(dbConnectionString);
                    connection.Open();

                    // Attempt to acquire the advisory lock with a timeout (Ensures only one process executes the DbUp migration if multiple processes are running simultaneously)
                    bool lockAcquired = false;
                    var startTime = DateTime.Now;

                    while ((DateTime.Now - startTime).TotalSeconds < 60) // Set a 60 seconds timeout for waiting for the lock
                    {
                        using var lockCommand = new NpgsqlCommand($"SELECT pg_try_advisory_lock({_appSettingModel.CurrentValue.PostgresqlAdvisoryLockKey});", connection);
                        var result = (bool)lockCommand.ExecuteScalar();

                        if (result)
                        {
                            lockAcquired = true;
                            break;
                        }

                        // Wait for 5 second before retrying
                        Thread.Sleep(5000);
                    }

                    if (lockAcquired)
                    {
                        try
                        {
                            var migrationUpgrader = DeployChanges.To
                                                    .PostgresqlDatabase(dbConnectionString)
                                                    .WithScriptsFromFileSystem(migrationPath)
                                                    .WithTransaction()
                                                    .WithExecutionTimeout(TimeSpan.FromMinutes(10))
                                                    .LogTo(dbUpLogger)
                                                    .Build();

                            if (!migrationUpgrader.IsUpgradeRequired())
                            {
                                _logger.LogInformationExtension("No pending migrations detected; skipping the migration process.");

                                // Run Data Seed Scripts
                                SeedData(dbConnectionString);
                            }
                            else
                            {
                                _logger.LogInformationExtension("New database migrations found; initiating the migration process.");

                                // Run Migration Scripts
                                var operation = migrationUpgrader.PerformUpgrade();
                                if (operation.Successful)
                                {
                                    _logger.LogInformationExtension("Database migration has been successfully completed.");

                                    // Run Data Seed Scripts Only if Migration Succeeds
                                    SeedData(dbConnectionString);
                                }
                            }
                        }
                        catch (Exception ex)
                        {
                            _logger.LogErrorExtension("An exception occurred during the database migration process.", ex);
                        }
                        finally
                        {
                            // Release the advisory lock after migrations/seeds are done
                            using var unlockCommand = new NpgsqlCommand($"SELECT pg_advisory_unlock({_appSettingModel.CurrentValue.PostgresqlAdvisoryLockKey});", connection);
                            unlockCommand.ExecuteNonQuery();
                            connection.Close();
                        }
                    }
                }
                else
                {
                    _logger.LogInformationExtension("No database migration script found; skipping the migration process.");
                }
            }
            return next;
        }

        private void SeedData(string dbConnectionString)
        {
            if (!string.IsNullOrEmpty(_appSettingModel.CurrentValue.Environment))
            {
                _logger.LogInformationExtension($"Seeding data for environment: {_appSettingModel.CurrentValue.Environment}");

                var dbUpLogger = new DbUpLoggerExtension(_logger);

                string seedPath = Path.Combine(AppDomain.CurrentDomain.BaseDirectory, "DbUpMigration", "Seeds", _appSettingModel.CurrentValue.Environment);

                if (Directory.Exists(seedPath) && Directory.EnumerateFiles(seedPath).Any())
                {
                    var seedUpgrader = DeployChanges.To
                                        .PostgresqlDatabase(dbConnectionString)
                                        .WithScriptsFromFileSystem(seedPath)
                                        .WithTransaction()
                                        .WithExecutionTimeout(TimeSpan.FromMinutes(10))
                                        .LogTo(dbUpLogger)
                                        .Build();

                    if (!seedUpgrader.IsUpgradeRequired())
                    {
                        _logger.LogInformationExtension("No new seed data script found; skipping the seeding process.");
                    }
                    else
                    {
                        var seedResult = seedUpgrader.PerformUpgrade();

                        if (seedResult.Successful)
                        {
                            _logger.LogInformationExtension($"Data seeding for {_appSettingModel.CurrentValue.Environment} completed successfully!");
                        }
                        else
                        {
                            _logger.LogErrorExtension($"An error occurred during data seeding for {_appSettingModel.CurrentValue.Environment}.", seedResult.Error);
                        }
                    }
                }
                else
                {
                    _logger.LogInformationExtension("No seed data script found; skipping the seeding process.");
                }
            }
        }
    }
}


```

```csharp

using DbUp.Engine.Output;
using PBR.Util.Logging;

public class DbUpLoggerExtension : IUpgradeLog
{
    public readonly ILogger _logger;

    public DbUpLoggerExtension(ILogger logger)
    {
        _logger = logger ?? throw new ArgumentNullException(nameof(logger));
    }

    public void WriteInformation(string format, params object[] args)
    {
        _logger.LogInformationExtension(string.Format(format, args));
    }

    public void WriteError(string format, params object[] args)
    {
        _logger.LogErrorExtension(string.Format(format, args), null);
    }

    public void WriteWarning(string format, params object[] args)
    {
        _logger.LogWarningExtension(string.Format(format, args));
    }
}


```

### 3. Integration in `Program.cs`

```csharp
// Add DbUp Migration
services.AddTransient<IStartupFilter, DatabaseMigrationInitFilter>();
```
This ensures that `DatabaseMigrationInitFilter` runs during application startup.

### 4. Migration Folder Structure
```
DbUpMigration
 ├── Migrations  (Contains migration scripts)
 │   ├── 20250212-111700-PBR-34-Initial-Schema.sql
 │   ├── 20250213-103000-PBR-35-Move-Detarment-To-Workspace.sql 
 ├── Rollbacks   (Contains rollback scripts)
 ├── Seeds       (Contains environment-specific seed data)
 │   ├── Dev
 │   │	├──	20250224-013700-PBR-34-Default-PowerBi-Account.sql
 │   ├── QA
 │   ├── UAT
```

## DbUp Execution Process with Multiple Iterations
DbUp ensures that database migrations execute sequentially while preventing duplicate execution of already applied scripts. The execution process follows these steps:

1. **Initialization and Configuration:**
   - The `DatabaseMigrationInitFilter` checks if `EnableDatabaseMigration` is set to `true` in the application settings.
   - The database connection string is retrieved.
   - The migration scripts directory is verified.

2. **Ensuring Database Existence:**
   - The database is created if it does not exist using `EnsureDatabase.For.PostgresqlDatabase(dbConnectionString);`.

3. **Advisory Lock for Concurrency Handling:**
   - The process attempts to acquire an advisory lock to prevent multiple instances from running migrations simultaneously.
   - If the lock is not acquired, it retries every 5 seconds for up to 60 seconds.

4. **Checking and Executing Migrations:**
   - DbUp checks if any new migration scripts exist.
   - If migration is required, the scripts are executed sequentially using `DeployChanges.To.PostgresqlDatabase(dbConnectionString).WithScriptsFromFileSystem(migrationPath).Build();`.
   - If migration succeeds, seed scripts are executed.
   - **Note:** All migration, rollback, and seed data scripts should be idempotent to avoid issues when executed multiple times.

5. **Handling Multiple Iterations:**
   - Each time the application starts, DbUp checks for new migration scripts.
   - Already applied scripts are skipped since DbUp tracks execution history in the database.
   - New scripts (added in subsequent deployments) are applied in order.
   - This ensures an incremental migration approach without reapplying old scripts.

6. **Releasing the Advisory Lock:**
   - Once migrations and seed scripts complete, the advisory lock is released.

This process guarantees that only new scripts are executed while maintaining consistency and preventing race conditions in multi-instance environments.

## Rollback Strategy Notes
1. Rollbacks are **manual** with DbUp (DbUp does not support automatic rollbacks).
2. Always test rollback scripts before deployment.
3. Ensure rollback scripts reverse the exact changes made in migration scripts.

## Advantages of Using DbUp

1. **Simplicity**
   - No complex ORM setup is required.
   - Migration scripts are plain SQL, making them easy to write and review.

2. **Transactional Migrations**
   - Ensures that database schema changes are atomic and consistent.

3. **Version Control Friendly**
   - Migration scripts are stored as files, allowing version control.

4. **Environment-Specific Seeding**
   - Enables different seed data for different environments (Dev, QA, UAT).

5. **Concurrency Handling**
   - Advisory locks prevent multiple instances from running migrations simultaneously.

6. **Logging Support**
   - Logs migration progress using `DbUpLoggerExtension`.

## Disadvantages of Using DbUp

1. **No Built-in Rollback Support**
   - DbUp does not provide automatic rollback functionality. Rollbacks must be managed manually via separate scripts in the `Rollbacks` folder.

2. **Sequential Execution Requirement**
   - Scripts execute in the order they are found in the directory; changing script execution order requires renaming files.

3. **Limited ORM Integration**
   - DbUp is not an ORM and does not integrate directly with Entity Framework.

4. **No Schema Diffing**
   - Unlike tools such as Liquibase or Flyway, DbUp does not generate schema diffs automatically.

## Conclusion
DbUp provides a lightweight, script-based approach to database migrations in .NET API projects. While it lacks built-in rollback and schema diffing features, it is simple, easy to integrate, and supports transactional execution, making it a reliable choice for managing database schema changes in database.
