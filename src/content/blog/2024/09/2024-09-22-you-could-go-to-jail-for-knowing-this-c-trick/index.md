---
title: "You could go to jail for knowing this C# trick"
date: 2024-09-22
categories: 
  - "c"
tags: 
  - "c"
  - "hacking"
  - "private-access-modifier"
  - "secure-code"
---

A common mistake among less experienced programmers (and even experienced ones) is to believe that the _private_ access modifier is enough to keep our code secure and prevent it from being accessed improperly.

![](images/image-1-1024x1024.png)

**This is a significant mistake!**

**Private methods and properties can be accessed through reflection!**

Yes, believe it or not, you can use sensitive code from many public libraries through reflection. This could be considered a hacking technique that you should be aware of. In this blog, we trust that our readers are ethically responsible individuals and we do not encourage the use of bad practices. However, we do encourage the correct use of techniques that help protect sensitive information in our libraries.

Consider this example.

We have a library with a public class like this

```
namespace Library;

public class PublicClass
{
    public void PublicMethod() {
        Console.WriteLine("You can have access to this method!");
    }

    private void PrivateMethod() { 
        Console.WriteLine("You can't access this method!");
    }
}
```

We want to protect our private method, so we declare it as private.

Even if the method is private we can call it externally with a code like this:

```
using System.Reflection;
using Library;
using ProtectedLibrary;

// Non protected
var flags = BindingFlags.NonPublic | BindingFlags.Instance;

var privateMethod = typeof(PublicClass)
    .GetMethod("PrivateMethod", flags);

var instance = new PublicClass();
privateMethod!.Invoke(instance, new object[]{});

// Output: You can't access this method!
```

Yes, this sucks, it really sucks! But what can we do? How can we ensure that our private method is truly private?

Here are some tips:

- Hide your concrete classes behind interfaces.

- Don't allow direct instantiation of your classes; use Factories or Builders.

- Declare your concrete classes with the _internal_ modifier.

## Hide your concrete classes behind interfaces

We can declare an interface with only the public method.

```
public interface IProtectedClass
{
    public void PublicMethod();
}

class ProtectedClass: IProtectedClass
{
    public void PublicMethod()
    {
        Console.WriteLine("You can have access to this method!");
    }

    private void PrivateMethod()
    {
        Console.WriteLine("You can't access this method!");
    }
}
```

We are going to expose the _IProtectedClass_ interface instead of _ProtectedClass_, so the _PrivateMethod_ is not visible.

## Don't allow direct instantiation of your classes; use Factories or Builders.

We are going to use a Factory to instantiate _ProtectedClass_, and we will not allow the client application to create the concrete class directly

```
public class Factory
{
    public static IProtectedClass GetProtectedClass()
    {
        return new ProtectedClass();
    }
}
```

## Declare your concrete classes with the _internal_ modifier.

Finally, we use the internal access modifier to make _ProtectedClass_ visible only within the current project.

```
internal class ProtectedClass: IProtectedClass
{
  // Code omitted
}
```

If we try to access the _PrivateMethod_ we will got the following errors  

```
var flags = BindingFlags.NonPublic | BindingFlags.Instance;

// var privateMethod = typeof(ProtectedClass) <- this cause compilation error
var privateMethod = typeof(IProtectedClass)
    .GetMethod("PrivateMethod", flags);

var instance = Factory.GetProtectedClass();
privateMethod!.Invoke(instance, new object[]{}); // privateMethod is null here
```

Links

- Github: [https://github.com/FractalCodeRicardo](https://github.com/FractalCodeRicardo)

- Medium: [https://medium.com/@nosilverbullet](https://medium.com/@nosilverbullet)

- Web page: [https://programmingheadache.com](https://programmingheadache.com/)

- Youtube: [https://www.youtube.com/@ProgrammingHeadache](https://www.youtube.com/@ProgrammingHeadache)

- Buy me a Coffee: [https://buymeacoffee.com/programmingheadache](https://buymeacoffee.com/programmingheadache)
