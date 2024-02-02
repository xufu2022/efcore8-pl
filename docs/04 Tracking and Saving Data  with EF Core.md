# Tracking and Saving Data with EF Core

## Overview

- DbContext and entity roles in tracking
- Tracking and saving workflow
- Inserting, updating, and deleting objects
- Tracking and saving multiple objects
- Bulk operation support
- How persistence differs in disconnected apps

## Tracking Components

- DbContext : Represents a session with the database
- DbContext.ChangeTracker :  Manages a collection of EntityEntry objects
- EntityEntry : State info for each entity: CurrentValues, OriginalValues, State enum, Entity & more

> Entity and the DbContext

- In-Memory Objects
- Entities :  In-memory objects with key (identity) properties that the DbContext is aware of
- DbContext : Contains EntityEntry objects with reference pointers to in-memory objects

## Understanding Tracking and Saving Workflow

Tracking and Saving Workflow

1. EF Core creates tracking object for each entity
1. DbContext maintains EntityEntries
1. On SaveChanges
1. Returns # Affected & new PKs
1. Updates entity PKs and FKS
1. Resets state info

## How DbContext Discovers About Changes

> DbContext.ChangeTracker.DetectChanges()

 Reads each object being tracked and updates state details in EntityEntry object

> DbContext.SaveChanges()

 Always calls DetectChanges for you

> You can call DetectChanges() in your code if needed


The entities are just plain objects and don’t communicate their changes to the DbContext.

## Various Paths for Tracking to Detect Edits

- If DbContextis aware of object, it will detect changes on SaveChanges
- DbSet and DbContext also have an Update() method
- More advanced: modify state directly

## Updating Untracked Objects

Begin Tracking and Set State to Modified

Track via DbSet : DbSet knows the type Knows right away that it’s Modified

Track via DbContext : DbContext will discover the type Knows right away that it’s Modified

## More Pointers for Disconnected Data

- Update is not typically needed when object is tracked
- Update and Add and Remove are important for untracked objects
- Your app logic will need to know when to call Update, Add or Remove


> Bulk Delete

 ExecuteDelete() for untracked data


## Batched Add/Update/Delete

- Min and Max driven by provider
- SQL Server minimum is 2, and maximum is 42
- Inserts return new key in OUTPUT
- Update & Delete return 1 in OUTPUT

```cs
public class SqlServerModificationCommandBatch :
 AffectedCountModificationCommandBatch
 {
 // https://docs.microsoft.com/sql/sql-server/maximum-capacity-specifications-for-sql-server
 private const int DefaultNetworkPacketSizeBytes = 4096;
 private const int MaxScriptLength = 65536 * DefaultNetworkPacketSizeBytes / 2;
 /// <summary>
 ///     The SQL Server limit on parameters, including two extra parameters to 
///        
sp_executesql (@stmt and @params).
 /// </summary>
 private const int MaxParameterCount = 2100 - 2;
```

Additional Batch Constraints of SQL Server Provider 

[Note] that min, max, etc.  are configurable

## Updating and Deleting Multiple Untracked Objects

QA: 

[Query].ExecuteDelete() //for untracked data

[Query].ExecuteUpdate()

## Executing Directly in the Database on Untracked Data

-  Delete (or update) a row when you only know its’ ID
-  Delete (or update) rows matching a certain criteria
-  Avoid wasting time and resources retrieving unneeded objects

> Execute Means “Do It Now”
- Executes right away
- Does not interact with ChangeTracker
- Returns # of rows affected

ExecuteUpdate

```cs
//Set a simple value
 [Query].ExecuteUpdate(
 setters => setters.SetProperty(b => b.PropertyToChange, newvalue)
 );
//Set a value based on a property 
[Query].ExecuteUpdate(
 setters => setters.SetProperty(b => b.PropertyToChange,b=>Transform(b.Property))
 );
```

**Do not mix tracking and these Execute methods!**

## Review

ChangeTracker and SaveChanges provides a lot of benefits

- An entity is unaware of the change tracker
- Database provider works with EF Core to send correct and efficient commands
- Batches insert, update, and delete commands to reduce roundtrips
- Determines if an explicit transaction is needed or not
- DB generated values returned efficiently and propagated back into entities


