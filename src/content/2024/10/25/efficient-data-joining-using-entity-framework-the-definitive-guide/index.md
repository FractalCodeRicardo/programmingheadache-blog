---
title: "Efficient Data Joining using Entity Framework: The Definitive Guide"
date: 2024-10-26
categories: 
  - "c"
  - "entityframework"
image: "/images/efficient-image-5-1024x1024.png"
---

In the C# ecosystem, there are several ways to join data from tables in a database, one of the most popular approaches is using Code First with Entity Framework.

![](/images/efficient-image-5-1024x1024.png)

Joining data between tables is a trivial task in most applications. The problem arises when the database starts to grow, and if the correct approach is not followed, it can lead to bottlenecks in the application, unnecessary resource consumption, and a poor user experience.

This article measures the execution time of some of the most common ways to perform table joins using Entity Framework.

## Scenario

Consider this typical table structure in a sales system, where we have a Product table and a Category table.

![](/images/efficient-image-2.png)

Using Code First - EF we have something like this

```csharp
public class Category
{
    public int CategoryId { get; set; }
    public string Name { get; set; } = string.Empty;
}

public class Product
{
    public int ProductId { get; set; }
    public string Name { get; set; } = string.Empty;
    public decimal Price { get; set; }

    public int CategoryId { get; set; }

    [ForeignKey("CategoryId")]
    public Category Category { get; set; }
}
```

We want to retrieve all products along with their category information.

## Approaches

**1\. Navigation property**

This is the recommended way to do it nowadays and it consists in using the include method to fill the Category property.

```csharp
var context = new Context();
var products = context.Products
        .Include(i => i.Category)
        .ToList();
```

**2\. Retrieve Category manually**

In some cases, when we don't have the navigation property and, if the database is small, we can use this alternative approach:

```csharp
var context = new Context();
var products = context.Products
        .ToList();

foreach (var product in products)
{
    var category = context
        .Categories
        .FirstOrDefault(i => i.CategoryId == product.CategoryId);
}
```

**3\. Retrieve Category manually with caching**

We can reduce the number of database calls by implementing a caching mechanism. A straightforward approach is to use a dictionary:

```csharp
var context = new Context();
var cache = new Dictionary<int, Category>();

var products = context.Products
        .ToList();

foreach (var product in products)
{
    var categoryId = product.CategoryId;

    // We only made the database call if the category is not cached
    if (!cache.ContainsKey(categoryId))
    {
        cache[categoryId] = context
                  .Categories
                  .Where(i => i.CategoryId == categoryId)
                  .First();
    }

    var category = cache[categoryId];

}
```

**4\. Retrieve all the categories we need in one call**

Instead of making multiple database calls, we can try to retrieve all the data at once (this have restrictions in some database engines)

```csharp
var context = new Context();
var products = context.Products
        .ToList();

// We create a list of ids to retrieve
var idsToRetrieve = products
    .Select(i => i.CategoryId)
    .Distinct()
    .ToList();

// We get all categories we need    
var categories = context
  .Categories
  //internally this creates an SQL In sentence. SELECT * FROM Categories WHERE id IN(1, ..., N)
  .Where(i => idsToRetrieve.Contains(i.CategoryId)) 
  .ToDictionary(i => i.CategoryId, i => i);
  

foreach (var product in products)
{
    var category = categories[product.CategoryId];
}
```

**5\. SQL like syntax**

We can use SQL like syntax that internally will cross the data using SQL

```csharp
var context = new Context();
var res = from product in context.Products
          join category in context.Categories
          on product.CategoryId equals category.CategoryId
          select new
          {
              product = product,
              category = category
          };

var list = res.ToList();
```

**6\. Raw SQL**

We can use an raw SQL query using SqlQuery method

```csharp
var context = new Context();
var query = context.Database
                .SqlQuery<ProductDto>(
                    $"""
                    SELECT 
                        p.ProductId,
                        p.Name,
                        p.Price,
                        c.CategoryId,
                        c.Name as CategoryName
                    FROM 
                    Products p 

                    JOIN Categories c 
                        ON p.CategoryId = c.CategoryId
                    """)
                .ToList();
```

## Performance of each approach

To measure the performance of each approach, some data is inserted into the products and categories tables. Then, the data is gradually increased.

![](/images/efficient-image-3-1024x463.png)

From this graph, we can see that the Manual approach is the slowest, and SQL is the fastest. It is clear that manual approach must be discarded when we have big data.

Lets see the other approaches in more detail

![](/images/efficient-image-4-1024x463.png)

The SQL approach is the best option when performance is a priority. However, in many cases, we don't want SQL strings in our code, so the next best options are the All-in-One or Include approaches. The Include approach is easy to use and offers good performance. Manual Caching and All-in-One are also good options, though they have slower performance.

## Conclusions

- Avoid the Manual approach if your database starts to grow.

- If performance is a high priority, use the SQL approach.

- If your models start to become a code smell due to too many navigation properties, use the All-in-One approach.

- If you have clean models, use the Include approach.

- If you need to join more than two tables, consider using the SQL-Like Join.

## Source Code

If you want to do your own tests, here is the code:

[https://github.com/FractalCodeRicardo/programmingheadache-misc/tree/main/eficient-ways-joining](https://github.com/FractalCodeRicardo/programmingheadache-misc/tree/main/eficient-ways-joining)

Links

- Github: [https://github.com/FractalCodeRicardo](https://github.com/FractalCodeRicardo)

- Medium: [https://medium.com/@nosilverbullet](https://medium.com/@nosilverbullet)

- Web page: [https://programmingheadache.com](https://programmingheadache.com/)

- Youtube: [https://www.youtube.com/@ProgrammingHeadache](https://www.youtube.com/@ProgrammingHeadache)

- Buy me a Coffee: [https://buymeacoffee.com/programmingheadache](https://buymeacoffee.com/programmingheadache)
