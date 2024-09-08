# Database Migrations & Rollback Strategies

## 1. **Versioning Scripts (Manual or Tool-Based)**

### **Migration Strategy:**
- The **versioning scripts approach** involves creating separate SQL script files for each schema or data change. These scripts are ordered by a version number or timestamp and applied in sequence during deployment.
  
- **Manual Migration**:
  - Each change is saved in a separate SQL script file, named sequentially. Example naming conventions might include:
    - `001_AddCustomersTable.sql`
    - `002_AddOrdersTable.sql`
  
  - These files contain DDL statements such as `CREATE TABLE`, `ALTER TABLE`, and data migration statements like `UPDATE` or `INSERT`.
  
  - To execute these scripts, you either run them manually or use tools like **DbUp**, **Flyway**, or **Liquibase**. These tools automatically track which scripts have been applied by creating a table in the database to store the applied version numbers (e.g., `SchemaVersions` table). This prevents scripts from being reapplied during future deployments.

- **Version Management**:
  - Scripts must be applied in the correct order. Version control systems (e.g., Git) ensure the development team can track changes and apply them consistently across different environments (dev, test, production).
  
  - Some migration tools enforce a strict order of execution, rejecting migrations that are out of sequence. For example, Flyway reads scripts by their version or timestamp prefix (`V1__Create_Customers.sql`).

- **Testing**:
  - Before running scripts in production, always test them in a staging environment to catch any issues. This helps verify the correct schema changes and data transformations without risking production data.

### **Rollback Strategy**:
- In a versioning system, each migration must have a corresponding rollback script. The rollback script essentially **reverts** the changes made by the migration script:
  - For example, if the migration script creates a new table, the rollback script should drop that table.
  - If the migration script adds a new column, the rollback script should drop that column.

- **Manual Rollback**:
  - Rollbacks are manually maintained in reverse order of migrations, ensuring that if a migration fails or introduces bugs, you can reverse it systematically.
  - Example:
    - Migration `001_AddCustomersTable.sql`: 
      ```sql
      CREATE TABLE Customers (Id INT PRIMARY KEY, Name NVARCHAR(100));
      ```
    - Rollback `001_Rollback_AddCustomersTable.sql`: 
      ```sql
      DROP TABLE Customers;
      ```

- **Tool-Based Rollback**:
  - Tools like Flyway or Liquibase may support rollback operations, enabling the use of commands to **undo migrations** automatically. However, you must explicitly write the rollback logic.
  
- **Considerations**:
  - Data loss: When dropping columns or tables, any data stored in them is lost. In such cases, manual backups or transformations may be necessary before rollback.
  - Complex rollbacks: Not all migrations are easily reversible. Data transformation scripts (e.g., `UPDATE`, `DELETE`) may require custom logic to restore previous values.

### **Pros**:
- Full control over each migration and rollback step.
- Transparency in changes: You can review each SQL script to see exactly what changes are being made.
- Flexible: Works with any database and requires no special framework.

### **Cons**:
- **Manual effort**: Each migration and rollback must be written and maintained carefully.
- Error-prone: Developers need to be disciplined in ensuring rollback scripts are created and accurate.
- Rollback complexity: Complex migrations involving data changes can be difficult to revert.

---

## 2. **Transactional Migrations**

### **Migration Strategy:**
- A **transactional migration** wraps the entire migration script inside a **database transaction**. This ensures that all changes in the script are applied as a single atomic operation. If any part of the migration fails, the entire transaction is rolled back, preventing the database from entering a partially migrated state.

- **What is a Transaction?**:
  - A **transaction** groups multiple SQL operations together into a single unit of work. Either all the operations are applied (committed), or none (rolled back).
  
  - Example of a transactional migration script:
    ```sql
    BEGIN TRANSACTION;

    ALTER TABLE Customers ADD Email NVARCHAR(255); -- Add new column
    UPDATE Customers SET Email = 'test@example.com' WHERE Name = 'John Doe'; -- Data transformation

    COMMIT; -- Apply all changes if successful
    ```

- **Why Use Transactions?**
  - Prevents **partial migrations**: If the migration fails midway, no changes are applied, ensuring data integrity.
  - Provides an automatic rollback mechanism: If an error occurs, all operations are reverted to the original state.

- **Tools**: Many migration tools like **DbUp** and **Flyway** support running migration scripts inside transactions. In case of an error, the tool will handle the rollback automatically.

### **Rollback Strategy**:
- Since the migration is inside a transaction, if the migration fails, the **rollback happens automatically** because the transaction is not committed.

- However, for successful migrations that need to be **reverted later** (e.g., a bug discovered post-deployment), a **manual rollback script** is required to undo the changes.

  - Example rollback for the script above:
    ```sql
    BEGIN TRANSACTION;

    ALTER TABLE Customers DROP COLUMN Email; -- Remove column
    UPDATE Customers SET Email = NULL WHERE Name = 'John Doe'; -- Optional: reverse data change

    COMMIT;
    ```

### **Pros**:
- **Atomicity**: The migration is applied as an all-or-nothing operation, ensuring the database remains in a consistent state.
- **Automatic rollback on failure**: If a migration fails, no changes are committed, reducing risk.

### **Cons**:
- **Not all databases support transactional DDL** (Data Definition Language) operations. For example, while databases like PostgreSQL and SQL Server support transactional DDL, some versions of MySQL do not.
- **Performance impact**: Long-running transactions can lock tables or rows, potentially degrading performance in production environments.

---

## 3. **Backward-Compatible Migrations (Zero-Downtime)**

### **Migration Strategy:**
- The **backward-compatible migration** approach ensures that the database schema changes do not break existing functionality. This allows you to run old and new versions of the application concurrently during the migration, minimizing risk and ensuring zero downtime.

- **Phased Migration**:
  - **Phase 1: Schema Expansion**: New schema elements (columns, tables, indexes) are added **without altering or removing existing elements**. The new schema is compatible with both the old and new versions of the application.
    - Example:
      ```sql
      ALTER TABLE Customers ADD Email NVARCHAR(255); -- New column for future use
      ```
  - **Phase 2: Application Update**: The new version of the application is deployed and starts using the new schema elements (e.g.,

 reading and writing to the `Email` column).
  - **Phase 3: Schema Cleanup**: Once the new schema is fully adopted and the old application version is no longer in use, any **obsolete schema elements** (e.g., old columns or tables) can be safely removed.

### **Rollback Strategy**:
- Since changes are backward-compatible, the **rollback** can often be handled by simply reverting the application to its previous version. The old schema elements remain intact until the final cleanup phase.

- If the schema needs to be reverted as well (e.g., due to bugs), a **manual rollback** script must be provided to remove or alter the changes introduced in the migration.

- Example rollback for the schema expansion:
  ```sql
  ALTER TABLE Customers DROP COLUMN Email; -- Remove new column
  ```

### **Pros**:
- **Zero downtime**: No need to take the database offline during migration.
- **Backward compatibility**: The old version of the application can run alongside the new version, minimizing risk.
- **Gradual transition**: Allows for a phased rollout of new features.

### **Cons**:
- **Slower migration**: This method often requires multiple deployment steps, as old schema elements can only be removed after the application has fully transitioned.
- **Temporary complexity**: Both old and new versions of the schema may need to coexist for some time, adding complexity to the codebase.

---

## 4. **Database Branching with Feature Flags**

### **Migration Strategy:**
- **Feature flags** are used to control access to new features or schema changes. With this approach, the migration is deployed first, but the application only begins using the new schema when the feature flag is enabled.

- **Steps**:
  1. Deploy the new schema changes (e.g., adding a new column or table).
  2. Modify the application to use the new schema, but keep the changes behind a **feature flag**. This flag is toggled to control access to the new feature.
  3. Gradually enable the feature flag in production, testing the new schema with a subset of users. If issues arise, the flag can be disabled, rolling back the application to the old schema.
  
  - Example feature flag logic:
    ```csharp
    if (FeatureFlags.IsEnabled("UseNewSchema"))
    {
        // Use new column Email
        var email = customer.Email;
    }
    else
    {
        // Fallback to old logic
        var email = customer.ContactEmail;
    }
    ```

### **Rollback Strategy**:
- **Immediate rollback**: If a problem occurs after enabling the feature, simply **disable the feature flag**, returning the application to its old behavior.
  
- Rollback of the database schema itself is not always necessary immediately, as the feature flag controls the application’s access to the new schema elements.
  
- Once the feature is deemed stable, you can fully enable the flag for all users. At this point, if needed, a rollback script can be written for the schema.

### **Pros**:
- **Granular control**: You can selectively enable or disable new schema changes in production.
- **Immediate rollback**: The feature flag provides a quick rollback mechanism without needing to alter the database.

### **Cons**:
- **Added complexity**: Managing feature flags and ensuring the application handles both old and new schema versions can be challenging.
- Requires coordination between the **application code** and the **database schema**, which may lead to additional technical debt if feature flags are not cleaned up after use.
