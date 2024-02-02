# Using EF Core to Query a Database

Overview

- Understand EF Core’s query workflow Basics of querying using EF Core and LINQ
- Filtering, sorting, and aggregating in queries
- Explore the SQL queries that EF Core is building for you
- Improve query performance by deactivating tracking when not needed

> Query Workflow

context.Authors.ToList()

1.  Express and execute query
1.  EF Core reads model, works with provider to work out SQL
1.  Sends SQL to database
1.  Receives tabular results
1.  Materializes results as objects
1.  Adds tracking details to DbContext instance

A LINQ execution method, e.g., ToList(), triggers the query to execute on the database.


## Finding an Entity Using its Key Value

- **DbSet.Find(keyvalue)**
- This is the only task that Find can be used for
- Not a LINQ method
- Executes immediately
- If key is found in change tracker, avoids unneeded database query

## QA:

Count() vs LongCount()

1. Count():
Return Type: int

Use Case: Count() is used to get the count of elements in a sequence when you expect the number of elements to fit within the range of an int, which is a 32-bit signed integer.

Limitation: If the count exceeds the maximum value of int (2,147,483,647), it will result in an overflow exception, potentially leading to incorrect results.

2. LongCount():
Return Type: long

Use Case: LongCount() is used when you need to get the count of elements in a sequence, and you want to ensure that the count is represented as a 64-bit signed integer (long). This method is suitable when dealing with very large datasets where the count may exceed the range of int.

Benefit: It can handle larger counts without causing overflow exceptions.

In most cases, using Count() is sufficient and efficient because it returns an int, which is typically large enough to represent the number of elements in most collections. However, if you are dealing with very large datasets or need to ensure compatibility with extremely high counts, then LongCount() should be used to prevent overflow issues.

## Aggregate Method Pointers

 Last methods require query to have an OrderBy() method otherwise will throw an exception

 The need to call OrderBy() or ensure a specific order primarily applies when working with collections that don't have an inherent order or when you need to retrieve the last element based on a specific ordering criterion. For a simple list of integers like in your example, it is not necessary to use OrderBy().

```cs
class Program
{
    static void Main()
    {
        List<int> numbers = new List<int> { 10, 30, 20, 40, 50 };
        
        //n this specific case where you have a list of integers, the Last() method will work without explicitly calling OrderBy().
        int lastNumber = numbers.Last();
        // Order the list in ascending order
        List<int> orderedNumbers = numbers.OrderBy(x => x).ToList();

        // Use LastOrDefault on the ordered list
        int lastOrDefaultNumber = orderedNumbers.LastOrDefault();

        Console.WriteLine("The last number in the ordered list is: " + lastOrDefaultNumber);
    }
}

```

## Change tracking is expensive

```cs
protected override void OnConfiguring (DbContextOptionsBuilder optionsBuilder)
 {
 optionsBuilder.UseSqlServer(myconnectionString).UseQueryTrackingBehavior(QueryTrackingBehavior.NoTracking);
 }
```

## Review

- EF Core, with provider’s help, transforms LINQ queries into SQL
- Query execution is triggered with specific methods
- Workflow includes sending SQL to database 
- EF Core materializes database results into objects
- LINQ filter, sorting and aggregating methods are also translated into SQL
- You can improve performance by disabling the default tracking when not needed