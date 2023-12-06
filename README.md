# Creating Deployment & Rollback SQL Scripts from Entity Framework Core migrations

Entity Framework Core (EF Core) is an Object-Relational Mapper (ORM) for .NET. It simplifies database interactions by allowing developers to work with data using .NET objects, thereby abstracting the complexities of working directly with database queries. EF Core is particularly useful for managing data models and database schemas, ensuring synchronization between the application’s data model and the underlying database structure. Its migrations feature provides a way to incrementally update the database schema to keep it in sync with the application's data model while preserving existing data.

Deploying EF Core migrations can follow various strategies, with some approaches being more suitable for production environments and others for the development lifecycle. One critical aspect of migrations is database schema versioning, which helps maintain consistency and predictability across different environments.

According to Microsoft's [EF Core documentation](https://docs.microsoft.com/en-us/ef/core/managing-schemas/migrations/applying?tabs=dotnet-core-cli), the recommended approach for deploying migrations to a production database is by generating SQL scripts. The advantages of this strategy include:

- **Review for Accuracy**: SQL scripts can be thoroughly reviewed before applying, which is important to mitigate risks involved in schema changes, especially in production databases where incorrect changes can lead to data loss.
- **Customization**: Scripts can be modified as needed to cater to specific requirements of a production database.
- **Deployment Integration**: SQL scripts can be integrated with deployment technologies and can form a part of Continuous Integration (CI) processes.
- **DBA Collaboration**: Scripts can be provided to Database Administrators (DBAs) for a controlled deployment process, allowing for expert oversight and archival.

In the following sections, we will delve into the process of generating SQL scripts for applying and rolling back EF Core migrations, ensuring a smooth and reliable update to your database schema.

## Generating SQL scripts for applying migrations

To generate SQL scripts for applying migrations using the .NET Core CLI, use the following command. It generates a script to transition a blank database to the latest migration:

```bash
dotnet ef migrations script
```

This command can be customized with two optional parameters to specify a range of migrations:

- **From Migration**: This should be the last migration applied to the database before running the script. If no migrations have been applied, specify `0` (this is the default).
- **To Migration**: This represents the last migration that will be applied to the database after running the script. By default, it is set to the latest migration in your project.

To create a script starting from a specific migration to the latest, use:

```bash
dotnet ef migrations script FromMigrationName
```

For generating a script between two specific migrations, include both the `from` and `to` migration names:

```bash
dotnet ef migrations script FromMigrationName ToMigrationName
```

## Generating Rollback SQL Scripts for your migrations

Whenever you deploy changes to critical environments such as UAT or Production, it is always necessary to have a rollback script ready in case there are some issues faced during deployment and you need to rollback the changes so that the user experience/testing is not impacted.

You can generate rollback scripts using the same command, with the only difference being the inversion of the `from` and `to`.

For example, if your command for generating SQL scripts for applying migrations looks like the one below:

```bash
dotnet ef migrations script ThirdMigrationName FifthMigrationName
```

In this case, the command to generate the Rollback SQL Script would be as follows:

```bash
dotnet ef migrations script FifthMigrationName ThirdMigrationName
```

To rollback all migrations, you can specify `0` as the `to` argument, and the `from` argument would contain the migration from which you want the rollback script to start.

## Idempotent SQL scripts

EF Core can generate **idempotent** scripts, which contain logic to check which migrations have been applied (using the migrations history table) and only apply missing ones. This is useful if you don't know exactly what the last migration applied to the database was, or if you are deploying to multiple databases that may each be at a different migration.

To generate an idempotent script, use:

```bash
dotnet ef migrations script --idempotent
```

It is a good practice to always generate idempotent scripts so that you don't end up adding the same migration multiple times on your database leaving your database in an inconsistent state.

Before deploying to production the DBA should always review the script for accuracy. This is to ensure that the SQL script is updated in case the production database might have some minor changes which are not present in the other environments and a DBA might be aware of these changes.

## Example: Generating a Rollback Script

Let's consider a practical example of generating a rollback script for specific migrations in a .NET Core project using EF Core.

| MigrationId                            | ProductVersion |
|----------------------------------------|----------------|
| 20230505162732_AlterSPCreateTenantStoredProcedure | 6.0.3          |
| 20230504075816_AddNotificationMessage   | 6.0.3          |
| 20230424104102_OrderFillTable           | 6.0.3          |
| 20230713164744_AddOfferUpdatedEmployeeRelationship | 6.0.3          |
| 20230713165206_AddContractTableRelationship | 6.0.3          |

In this scenario, we want to roll back the last two migrations: **20230713164744_AddOfferUpdatedEmployeeRelationship** and **20230713165206_AddContractTableRelationship**.

The command to generate the rollback script would be:

```bash
dotnet ef migrations script 20230713165206_AddContractTableRelationship 20230424104102_OrderFillTable --project YourProjectPath --startup-project YourStartupProjectPath --output "YourOutputPath\rollback.sql" --context YourDbContext --verbose
```

This command generates a script to roll back the database to the state just after **20230424104102_OrderFillTable** was applied, effectively undoing the last two migrations.