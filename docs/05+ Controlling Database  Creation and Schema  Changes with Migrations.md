# Controlling Database Creation and Schema Changes with Migrations

## Overview
- Overview of EF Core Migrations API
- Setting up your project and Visual Studio for migrations
- Create and inspect a migration file
- Using EF Core Migrations to create a database or database scripts
- Reverse engineer an existing database into classes and DbContext

## Understanding EF Core Migrations

EF Core Needs to Comprehend the DB Schema

- Build SQL from your LINQ queries
- Materialize query results into objects
- Build SQL to save data into database

Mapping Your Data Model to the Database

- DbContext
- Conventional and custom mappings
- Database schema

EF Core Basic Migrations Workflow

**Define/Change Model >> Create  Migration File >>  Apply Migration to DB or Script**

## Getting and Understanding the Design-Time Migrations Tools

Creating and executing migrations happens at  design time

Migration Commands Lead to Migrations APIs

- PowerShell migrations commands : **add-migration**
- dotnet CLI migrations commands : **dotnet ef migrations add**

Migration Commands Lead to Migrations APIs


 1. Microsoft.EntityFrameworkCore.Design
 1. Microsoft.EntityFrameworkCore.Relational
- PowerShell migrations commands : **Microsoft.EntityFrameworkCore.Tools**
- dotnet CLI migrations commands : **dotnet tool install dotnet-ef**


> Using Migrations in dotnet CLI

- Add EF Core tools to system*
- Add EF Core Design** package to executable project
- Use “dotnet ef” commands at command line

## Add-Migration Tasks for Model Changes

1. Read DbContext to determine data model  ...and any seeding code
1. Compare data model to snapshot  ...and seeding instructions
1. Generate migration file to apply the deltas
1. Create updated snapshot file
 
 
Some EF Core Mapping Conventions (Defaults)

- DbSet name is the table name
- Class property name is the column name
- Strings are determined by provider  e.g., SQL Server: nvarchar(max)
- Decimals are determined by provider  e.g., SQL Server: decimal(18,2)
- “Id” or “[type]Id” are primary keys

Maps Strings to Non-Nullable Columns by Default

```cs
//sql server
firstname=table.column<string>(type:"nvarchar(max)", nullable: false)
//sqllite
firstname=table.column<string>(type:"TEXT", nullable: false)

```

## Using Migrations to Script or Directly Create the Database

-  Reads migration file
-  Generates SQL
-  Default: Displays SQL in editor
-  Use parameters to target file name etc

## Seeding a Database via Migrations

```cs

 modelBuilder.Entity<EntityType>().HasData(parameters)
 modelBuilder.Entity<Author>().HasData(new Author {Id=1, FirstName=“Julie”, .. };
```

### Specify Seed Data with ModelBuilder HasData Method
- Provide all non-nullable parameters including keys and foreign keys
- HasData will get translated into migrations
- Inserts will get interpreted into SQL
- Data will get inserted when migrations are executed

Add-Migration Tasks for Model Changes

Seeding with HasData will not cover all use cases for seeding

- Mostly static seed data
- Sole means of seeding
- No dependency on anything else in the database
- Provide test data with a consistent starting point

HasData will also be recognized and applied by EnsureCreated().

Scripting Multiple Migrations

Scripting migrations requires more control, so it works differently than update-database.

Some Scripting Options


