---
title: "Stop Messing Up Your Production Environment with This Essential C# Trick"
date: 2024-12-08
categories: 
  - "c"
---

When companies fail to properly manage their production environments, serious issues can arise, such as inserting test data into real databases, conducting transactions with customer information without consent, or altering legitimate balances that should not be modified, among other critical mistakes.

![](images/image-1024x1024.png)

Even large corporations, which dedicate significant resources to protecting their production environments, occasionally face incidents caused by mixing test and production environments. These errors can have significant operational and legal consequences.

Within the .NET environment, there is a preprocessor directive that helps you control whether the code is being debugged or if it is already a release. This can be used to add extra validations when working with production and test variables.

The idea is very simple. For example

```csharp
#if DEBUG
  // I'm in my local enviroment
#else
  // I'm in production
#endif
```

Suppose you have an application that uses Entity Framework. A simple approach to use the correct connection string can be as follows

```csharp
public class AppDbContext : DbContext
{

    protected override void OnConfiguring(DbContextOptionsBuilder builder)
    {
        var configuration = new ConfigurationBuilder()
             .AddJsonFile("appsettings.json")
             .Build();

        string connection = "";

        #if DEBUG
        connection = configuration.GetConnectionString("TestConnection");
        #else
        connection = configuration.GetConnectionString("ProductionConnection");
        #endif

        builder.UseSqlServer(connection);
    }
}
```

Suppose you are familiar with well-known conventions for your databases, for example: Sales for production and Sales\_Dev for local development. You can add extra validations like this:

```csharp

    protected override void OnConfiguring(DbContextOptionsBuilder builder)
    {
        var configuration = new ConfigurationBuilder()
             .AddJsonFile("appsettings.json")
             .Build();

        string connection = configuration.GetConnectionString("DefaultConnection")

        #if DEBUG
        
            if (!connection.Contains("_Dev")
            {
              throw new Exception("You are not using a development database");
            }

        #endif
        builder.UseSqlServer(connection);
    }
}
```

Another approach to controlling the production environment is through the use of environment variables. For example, when using ASP.NET applications, you can set the **ASPNETCORE\_ENVIRONMENT** variable which determines the connection string to be used.

Inside the application, you can access the value of the variable as demonstrated below:

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

if (app.Environment.IsProduction())
{
    // ASPNETCORE_ENVIRONMENT is set to Production
}

if (app.Environment.IsDevelopment())
{
    // ASPNETCORE_ENVIRONMENT is set to Development
}
```

So, you can validate if you are in the correct environment like this

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

#if DEBUG
if (app.Environment.IsProduction())
{
    throw new Exception("You are using the wrong enviroment");
}
#endif 
```

**Links**

- Github: [https://github.com/FractalCodeRicardo](https://github.com/FractalCodeRicardo)

- Medium: [https://medium.com/@nosilverbullet](https://medium.com/@nosilverbullet)

- Web page: [https://programmingheadache.com](https://programmingheadache.com/)

- Youtube: [https://www.youtube.com/@ProgrammingHeadache](https://www.youtube.com/@ProgrammingHeadache)

- Buy me a Coffee: [https://buymeacoffee.com/programmingheadache](https://buymeacoffee.com/programmingheadache)
