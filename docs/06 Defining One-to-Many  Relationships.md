# Defining One-to-Many Relationships

Overview

- Install a tool to visualize the data model
- How EF Core interprets various one-to many setups
- Benefits of foreign key properties
- Modify the entity classes
- Seed related data
- Use migrations to evolve the database to match a modified model
- Using mappings to guide EF Core when needed

## Visualizing EF Core’s Interpretation of Your Data Model

Shadow Properties : Properties that existing the data model but not the entity class. These 
can be inferred by EF Core or you can explicitly define them in the DbContext

Relationship Terminology

- Parent / Child
- Principal / Dependent

```cs
// Reference from parent to child is sufficient
 public class Author
 {
    public int Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public List<Book> Books { get; set; } // List<Child> in Parent
 }
 // This Child has no references back to Parent
 // Foreign Key (e.g, AuthorId) will be inferred in database

 public class Book
 {
    public int Id { get; set; }
    public string Title { get; set; }
 }

```

```cs
// Child has navigation prop pointing to Parent
 public class Author
 {
    public int Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public List<Book> Books { get; set; } // List<Child> in Parent
 }
// This child has a reference back to parent aka  “Navigation Property”
// It already understands the one-to-many because of the parent’s List<Child> 
// The reference back to parent aka  “Navigation Property” is a bonus 
// AuthorId FK will be inferred in the database

 public class Book
 {
    public int Id { get; set; }
    public string Title { get; set; }
    public Author Author { get; set; }
 }

```

```cs
// FK is recognized because of reference from parent
public class Author
 {
    public int Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public List<Book> Books { get; set; } // List<Child> in Parent
 }
 public class Book
 {
    public int Id { get; set; }
    public string Title { get; set; }
    public int AuthorId {get; set; }
 }
// AuthorId is recognized as foreign key for two reasons:
//  1) the known relationship from parent
//  2) follows FK naming convention (type + Id)

```

```cs
// Child has navigation prop pointing to Parent & an FK
public class Author
 {
    public int Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public List<Book> Books { get; set; } // List<Child> in Parent
 }

 public class Book
 {
    public int Id { get; set; }
    public string Title { get; set; }
    public Author Author { get; set; }
    public int AuthorId {get;set;}
 }
 // One to many is known because of List<Child>
 // Navigation and FK are bonus

```

```cs
// No detectable relationship
public class Author
 {
    public int Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
 }

 public class Book
 {
    public int Id { get; set; }
    public string Title { get; set; }
    public int AuthorId {get;set;}
 }
 //  This AuthorId property is just a random integer 
```

## Configuring a One-to-Many

```cs
// When convention can’t discover it because there are no references
protected override void OnModelCreating(ModelBuilder modelBuilder)
 {
    modelBuilder.Entity<Author>()
    .HasMany<Book>()
    .WithOne();
 }

```

Considering the Parent Navigation Property

- Coming from earlier versions? : EF Core is now smarter about missing navigation data, but not in all scenarios.
- Tip: Default to no navigation property : Only add it if your biz logic requires it!

Why have FK property?

```cs
// Only a navigation property to Author. You must have an author in memory to connect a book.
//  author.Books.Add(abook)
//  abook.Author=someauthor

public class Book
 {
    public int Id {get;set;}
    public string Title {get;set;}
    public Author Author{get;set;}
 }

//  With a foreign key property, you don’t need an Author object.
//   book.AuthorId=1
 public class Book
 {
    public int Id {get;set;}
    public string Title {get;set;}
    public Author Author{get;set;}
    public int AuthorId {get;set;}
 }

 //  ...and you can even eliminate the navigation property if your logic doesn’t need it
 public class Book
 {
    public int Id {get;set;}
    public string Title {get;set;}
    public int AuthorId {get;set;}
 }

```

> Foreign Key Properties & HasData Seeding

 HasData requires explicit primary and foreign key values to be set. With AuthorId now in Book, you can seed book data as well

 ```cs
 modelBuilder.Entity<Author>().HasData(new Author {Id=1, FirstName=“Julie”, .. };
 modelBuilder.Entity<Book>().HasData( new Book {BookId=1,AuthorId=1,Title=“Programming Entity Framework”});
```

 Seeding Related Data

- Provide property values including keys from  and foreign keys from “Parent”
- HasData will get interpreted into migrations
- Inserts will get interpreted into SQL
- Data will get inserted when migrations are executed 

[Notes] My Author & Book Key Properties Use Different Naming Conventions!

 That’s not a recommended coding practice. Let’s fix it.

Configuring a Non-Conventional Foreign Key 

FK is tied to a relationship, so you must first describe that relationship

 ```cs
 protected override void OnModelCreating(ModelBuilder modelBuilder)
 {
    modelBuilder.Entity<Author>()
    .HasMany(a => a.Books)
    .WithOne(b => b.Author)
    .HasForeignKey(b => b.AuthorFK);
 }
```

## Understanding Nullability and Required vs. Optional Principals

By default, every dependent must have a principal, but EF Core does not enforce this

 ```cs
 //Specify the FK property is nullable
 public class Book
 { ...
    public int? AuthorId { get; set; }
 }
// Map the FK property as not required
 modelBuilder.Entity<Author>()
    .HasMany(a => a.Books)
    .WithOne(b => b.Author)
    .HasForeignKey(b => b.AuthorId)
    .IsRequired(false);

// When FK property doesn’t exist, map the inferred (“shadow”) property as not required

 modelBuilder.Entity<Author>()
    .HasMany(a => a.Books)
    .WithOne(b => b.Author)
    .HasForeignKey("AuthorId")
    .IsRequired(false);
```

# Summary

- Many ways to describe one-to-many that conventions will recognize
- You can add mappings if those patterns don’t align with your desired logic
- Foreign keys are your friends
- An unconventional FK name is a common “gotcha” which you can fix in mappings
- Watch out for required principals!
- Use the EF Core Power Tools to verify how EF Core will interpret the data model
