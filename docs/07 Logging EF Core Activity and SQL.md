# Logging EF Core Activity and SQL

Overview

- Learn how EF Core extends .NET logging
- Adding EF Core logging to the app
- Many ways to filter what’s captured and format how it’s output
- Output to various targets

## Adding Logging to EF Core’s Workflow

EF Core’s logging is an extension of .NET Logging APIs

EF Core Logging Capabilities

EF Core captures:
- SQL
- ChangeTracker activity
- Interaction with database
- Database transactions

EF Core specific configurations:
- EnableDetailedErrors, EnableSensitiveData
- Filter based on message type (e.g., Database messages)
- Even more detailed filtering (see docs)


 ```cs
 // DbContextOptionsBuilder.LogTo Method
protected override void OnConfiguring (DbContextOptionsBuilder optionsBuilder)
 {
    optionsBuilder.UseSqlServer(someconnectionstring) .LogTo(target);
 }
```

Filtering Log Output with EF Core Message Categories and LogLevel

## .NET Logging API Configurations

Output targets : e.g., console, file
Log Levels : e.g., critical, debug


## Configuring LogLevel via .NET Logging API

 LogLevel enums
 - Debug (default)
 - Error
 - Critical
 - Information
 - Trace
 - Warning
 - None

## Even More Logging Features

-  Formatting
-  Detailed query error information
-  Filter on event types
-  What information goes in a log message
-  Show sensitive information e.g., parameters

 ```cs
 // LogTo Target Examples
optionsBuilder
 .LogTo(Console.WriteLine) //Delegate to Console.WriteLine


 private StreamWriter _writer = new StreamWriter (“EFCoreLog.txt", append: true); 

//  Delegate to StreamWriter.WriteLine. Be sure to dispose StreamWriter to save file
 optionsBuilder.LogTo(_writer.WriteLine)
 optionsBuilder.LogTo(log=>Debug.WriteLine(log));
```

# Encoding and Logging to the Console
Some encodings, e.g. code page 1256 for Arabic, won’t display in the VS console. If SQL contains one of these encodings, here’s a tweak to know

## Console Won’t Output Some Encodings

## Review

Logs are useful for debugging and learning
- EF Core logs provide more than just SQL
- You’ll probably want to use Simple Logging API (LogTo)
- EF Core logging extends .NET logging APIs
- Explicitly configure sensitive data in parameters to show in logs
- Many ways to customize and filter EF Core’s logging output
