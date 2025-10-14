---
title: "Must-Have Functions Every Repository Pattern Needs"
date: 2025-10-12
categories: 
  - "entityframework"
  - "programming"
tags: 
  - "code-smells"
  - "data-access-layer"
  - "dotnet"
  - "ef-tips"
  - "entity-framework-2"
  - "refactoring-tips"
  - "repository-pattern"
---

I’ve implemented the **Repository Pattern** in many projects since the early versions of **Entity Framework**, and I’ve noticed that introducing simple functions early can save hundreds — if not thousands — of lines of code in large applications.

![](images/imagen-576x1024.png)

In this article, I’ll show you a few **reusable Entity Framework functions** that you can use across your data access layer to make your code cleaner and more maintainable:

- Dynamically determine an entity’s primary key

- Delete an entity checking if it exists in the database

- Save (insert or update) an entity automatically without worrying about its state

Although these functions are straightforward to implement, many developers overlook them, leading to **duplicated code**, **complex code**, and even **code smells** that make maintenance harder over time

## Basic Repository pattern

A standard repository pattern in Entity Framework typically includes the following methods:

```
public interface IRepository<T> where T : class
{
  Task<IEnumerable<T>> GetAllAsync();
  Task<T?> GetByIdAsync(object id);
  Task AddAsync(T entity);
  void Update(T entity);
  void Delete(T entity);
  Task SaveChangesAsync();
}
```

###### Disadvantages of a standard repository pattern

- You need to know explicitly the id property, which makes it difficult to build truly generic operations that could reduce code duplication.

- Sometimes it’s necessary to first check if the entity exists before performing a delete.

- Save operations are divided into update or add, but most of the time we just want to persist an entity regardless of whether it already exists in the database

## Better Repository Pattern

## Dynamically determine the key

First we want to retrieve the ids dynamically. We are going to get the id values using reflection. First we get id properties

```
  private IEnumerable<PropertyInfo> GetIdsProperties()
  {
    var type = typeof(T);
    var properties = type.GetProperties();

    var ids = properties
      .Where(i => i.Name == "Id") //you can check if [Key] attribute is used
      .ToList();

    return ids;
  }
```

Then we get the id values using the properties

```
private object[] GetIds(T entity)
  {
    var type = typeof(T);

    var idsProperties = GetIdsProperties();
    var ids = new List<object>();

    foreach (var prop in idsProperties)
    {
      var value = prop.GetValue(entity);

      if (value == null)
        throw new Exception("Id can not be null");

      ids.Add(value);
    }

    return ids.ToArray();
  }
```

## Better deletes

I've seen many projects with this pattern used widely

```
using (var con = new AnimalsContext())
{
  var repo = new Repository<Animal>(con);

  var currentAnimal = await repo.GetByIdAsync(animalId);
  if (currentAnimal != null)
  {
    repo.Delete(currentAnimal);
    await repo.SaveChangesAsync();
  }
}
```

Even though this code can be written in a generic way, in practice you can see it repeated hundreds or even thousands of times in large projects.

Lets create a better delete that check if the data is present in the data base

```
  public async Task BetterDelete(params object[] ids)
  {
    var entity = await _dbSet.FindAsync(ids);

    if (entity != null)
    {
      _dbSet.Remove(entity);
    }
  }
```

Then the code is simplified like this

```
using (var con = new AnimalsContext())
{
  var repo = new Repository<Animal>(con);
  await repo.BetterDelete(animalId);
  await repo.SaveChangesAsync();
}
```

## Better save operations

Most of the time, we don’t care whether the data already exists in the database or not — we just want to save the entity.

In many projects, you’ll see this common pattern:

```
using (var con = new AnimalsContext())
{
  var repo = new Repository<Animal>(con);

  var currentAnimal = await repo.GetByIdAsync(animal.Id);
  if (currentAnimal == null)
  {
    // insert animal
    await repo.AddAsync(animal);
  }
  else
  {
    // update current animal
    currentAnimal.Age = animal.Age;
    currentAnimal.Name = animal.Name;
    currentAnimal.IsDomestic = animal.IsDomestic;

    repo.Update(currentAnimal);
  }

  await repo.SaveChangesAsync();
}
```

Let’s simplify this by creating a generic save function:

```
 public async Task SaveAsync(T entity)
  {
    var ids = GetIds(entity);
    var currentEntity = await _dbSet.FindAsync(ids);

    if (currentEntity == null)
    {
      await AddAsync(entity);
      return;
    }

    _context
      .Entry(currentEntity)
      .CurrentValues
      .SetValues(entity);
  }

```

And now the code becomes much simpler:

```
  using (var con = new AnimalsContext())
  {
    var repo = new Repository<Animal>(con);
    await repo.SaveAsync(animal);
    await repo.SaveChangesAsync();
  }
```

Conclusions

Even though these changes are relatively easy, if they aren’t applied early in development, the code base starts to become a code smell that will eventually require refactoring.

Don’t make that mistake, apply simple improvements like the ones above early on, and everything will go much smoother.

**Links**

- Github: [https://github.com/FractalCodeRicardo](https://github.com/FractalCodeRicardo)

- Medium: [https://medium.com/@nosilverbullet](https://medium.com/@nosilverbullet)

- Web page: [https://programmingheadache.com](https://programmingheadache.com/)

- Youtube: [https://www.youtube.com/@ProgrammingHeadache](https://www.youtube.com/@ProgrammingHeadache)

- Buy me a Coffee: [https://buymeacoffee.com/programmingheadache](https://buymeacoffee.com/programmingheadache)
