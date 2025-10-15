---
title: "Expert-Level C#: Advanced Tricks with Reflection and Generics"
date: 2024-09-01
categories: 
  - "c"
tags: 
  - "advanced"
  - "c"
  - "programming"
  - "tricks"
---

An advanced software developer uses generics and reflection to save themselves the work of writing hundreds of lines of code. There are a series of tricks that have been used for years, but unfortunately, they are not widely known. Here are some of those tricks.

![](images/image-1024x1024.png)

## 1\. Use lambda expression to get property names

The reason for doing this is simple. If you hard-code property names and later change them, you might break things without noticing. With this trick, you can safely change property names.

```csharp
// Instead of
// var nameProp = "Name"; 
// you can use 
var nameProp = PropertyName((Foo i) => i.Name);

public static string PropertyName<Entity, Prop>(
    Expression<Func<Entity, Prop>> expression)
{
    MemberExpression member = expression.Body as MemberExpression;

    return member.Member.Name;
}
```

## 2\. Map objects easily

Mapping common properties between objects is a frequent task in business software. By using generics and reflection, you can save a lot of time and code when performing this task.

Here is a simplified example of a mapper class  

```csharp

public class Mapper<TSource, TTarget>
    where TSource : class
    where TTarget : class
{
    public void Map(TSource source, TTarget target)
    {
        var typeSource = typeof(TSource);

        var properties = typeSource.GetProperties();

        // loop each property
        foreach (var sourceProperty in properties)
        {
            CopyValues(source, sourceProperty, target);
        }
    }

    public void CopyValues(TSource source, PropertyInfo sourceProperty, TTarget target)
    {
        var typeTarget = typeof(TTarget);
        var targetProperty = typeTarget.GetProperty(sourceProperty.Name);

        if (targetProperty == null) return;

        object value = sourceProperty.GetValue(source, null);
        targetProperty.SetValue(target, value, null);
    }
}
```

Now, instead of mapping manually all the common properties like this

```csharp
var foo = new Foo();
var bar = new Bar();

bar.Name = foo.Name;
bar.Name1 = foo.Name1;
bar.Name2 = foo.Name2;
bar.Name3 = foo.Name3;
```

You just call the mapper as shown above:

```csharp
var mapper = new Mapper<Foo, Bar>();
mapper.Map(foo, bar);
```

## 3\. Call private methods from outside their class

Believe it or not, even if a method is marked as private, it can still be called from outside its declaration. I’m not sure if this is a security flaw, but it’s something you should be aware of and consider other ways to protect your methods. That aside, you can use reflection to access and use these methods.

Consider this class with a private a method

```csharp
public class PrivateMethodClass 
{
    private void GetPrivateThings()
    {
        Console.WriteLine("This is private");
    } 
}
```

You can call the private method in this way

```csharp
var instance = new PrivateMethodClass();
var type = typeof(PrivateMethodClass);

var privateMethod = type.GetMethod("GetPrivateThings", BindingFlags.NonPublic | BindingFlags.Instance);

privateMethod.Invoke(instance, null);
```

## 4\. Collection Extensions

Yes, we love LINQ, and yes, you can easily extend it. For example, suppose you need a method that returns elements from a list between two specific dates.

You can do the following

```csharp
public static class CollectionExtensions
{
    public static IEnumerable<T> Between<T>(
        this IEnumerable<T> source, 
        Func<T, DateTime> getDate, 
        DateTime from,
        DateTime to)
    {

        return source.Where(entity => 
            getDate(entity) >= from && 
            getDate(entity) <= to);
    }
}
```

And use it like this

```csharp
var dates = new List<Foo>() {
    new Foo() { Date = new DateTime(2024, 1, 1)},
    new Foo() { Date = new DateTime(2024, 1, 2)}
};

var from = new DateTime(2024, 1, 1);
var to = new DateTime(2024, 1, 1);

var filtered = dates.Between(x => x.Date, from, to);
```

## 5\. Create complex expression in runtime

Suppose we want to filter a list with certain conditions

```csharp
var fooList = new List<Foo>() {
 // elements
};
 
var res = fooList
  .Where(i => i.Condition1 == "value1"
            && i.Condition2 == "value2"
            && i.Condition3 == "value2"
            && i.Condition4 == "value4"
        );
 
```

The problem with this, is that the filter condition Expression has to be written in compiled time. An alternative to create the condition in runtime is to build the Expression using reflection.

A common approach to build the expression is a class like the following

```csharp
public class ExpressionBuilder<T>
{

    // creates expresion entity => entity.propertyName == value
    public Expression CreatePropertyExpression(
      Expression entityParameter, 
      string propertyName, object? value)
    {
        var property = typeof(T).GetProperty(propertyName);
        var propertyAccess = Expression.MakeMemberAccess(entityParameter, property);
        var valueExpression = Expression.Constant(value, property.PropertyType);
        var equalsExpression = Expression.Equal(propertyAccess, valueExpression);

        return equalsExpression;
    }
    
     // creates expresion entity => entity.propertyName1 == value1
     //                            && entity.propertyName1 == value1
     //                            && entity.propertyNameN == valueN
    public Func<T, bool> ComplexExpression(
        List<Tuple<string, object>> conditions
        )
    {
        Expression expression = null;
        var parameter = Expression.Parameter(typeof(T), "entity");

        foreach (var condition in conditions)
        {
            var currentExpression = CreatePropertyExpression(
              parameter, 
              condition.Item1, 
              condition.Item2);

            if (expression == null)
            {
                expression = currentExpression;
                continue;
            }

            expression = Expression.AndAlso(expression, currentExpression);
        }

        var lambda = Expression.Lambda<Func<T, bool>>(expression, parameter);

        return lambda.Compile();
    }
}
```

And you can use it like this

```csharp
var builder = new ExpressionBuilder<Foo>();
   
// this can be create it at runtime     
var condition = new List<Tuple<string, object>>() {
   new Tuple<string, object>("Condition1", "Value1"),
   new Tuple<string, object>("Condition2", "Value2"),
     // etc
};

var expression = builder.ComplexExpression(condition);
var filtered = list.Where(expression).ToList();
```

You can see the code details here:

[https://github.com/FractalCodeRicardo/programmingheadache-misc/tree/main/advanced-tricks-reflection-csharp](https://github.com/FractalCodeRicardo/programmingheadache-misc/tree/main/advanced-tricks-reflection-csharp)

Links

## Links

  
Github: [https://github.com/FractalCodeRicardo](https://github.com/FractalCodeRicardo)

Medium: [https://medium.com/@nosilverbullet](https://medium.com/@nosilverbullet )

Web page: [https://programmingheadache.com](https://programmingheadache.com )

Youtube: [https://www.youtube.com/@ProgrammingHeadache](https://www.youtube.com/@ProgrammingHeadache)
