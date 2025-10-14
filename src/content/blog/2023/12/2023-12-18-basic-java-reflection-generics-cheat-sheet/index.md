---
title: "Basic Java Reflection / Generics cheat sheet"
date: 2023-12-18
categories: 
  - "java"
---

![](images/image-42.png)

Reflection is awesome it lets you handle class fields and methods dynamically, when using it, some of the common tasks that you will need are these:

1. Get Class<T> instance. This class has the information of a given type.

3. Create an instance of a given type.

5. Get fields of type.

7. Get a field by name

9. Set value of a private field

11. Set value of a private field using its setter method

13. Get value of a private field

15. Get value of a private field using getter method

17. Get methods of the type

19. Get method by name

21. Call method by name

We are going to use this class for the examples

```
public class Foo {

    private String bar;

    public String getBar() {
        return bar;
    }

    public void setBar(String bar) {
        this.bar = bar;
    }

    public void printBar() { 
        printBar("");
    }

    public void printBar(String param) {
        System.out.println(bar+param);
    }
}
```

**Get Class<T>**

```
Class<Foo> clazz = Foo.class;
```

**Create an instance**

```
 try {
 
      Class<Foo> clazz = Foo.class;
      Foo instance = clazz.getDeclaredConstructor().newInstance();
     
  } catch (Exception ex) {
      // handle exception
  }
```

**Get fields of type**

```
try {

    Class<Foo> clazz = Foo.class;
    Field[] fields = clazz.getDeclaredFields();

} catch (Exception ex) {
    // handle exception
}
```

**Get a field by name**

```
try {
    String name = "bar";
    Class<Foo> clazz = Foo.class;
    Field field = clazz.getDeclaredField(name);

} catch (Exception ex) {
    // handle exception
}
```

**Set value of a private field**

```
try {
    String name = "bar";
    Class<Foo> clazz = Foo.class;
    Foo instance = clazz.getDeclaredConstructor().newInstance();
    
    Field field = clazz.getDeclaredField(name);
    field.setAccessible(true);
    field.set(object, value);

} catch (Exception ex) {
    // Handle exceptions
}
```

**Set value of a private field using its setter method**

```
try {

    String name = "bar";
    Class<Foo> clazz = Foo.class;
    Foo instance = clazz.getDeclaredConstructor().newInstance();
    
    String fieldName = name.substring(0, 1).toUpperCase() + name.substring(1);
    String setMethod = "set" + fieldName;
    Method method = clazz.getMethod(setMethod);

    method.invoke(instance, value);

} catch (Exception ex) {
    // Handle exceptions
}
```

**Get value of a private field**

```
try {
    String name = "bar";
    Class<Foo> clazz = Foo.class;
    Foo instance = clazz.getDeclaredConstructor().newInstance();

    Field field = clazz.getDeclaredField(name);
    field.setAccessible(true);

    Object value = field.get(instance);
    
} catch (Exception ex) {
    // Handle exceptions
}
```

**Get value of a private field using getter method**

```
try {
    String name = "bar";
    Class<Foo> clazz = Foo.class;
    Foo instance = clazz.getDeclaredConstructor().newInstance();

    String fieldName = name.substring(0, 1).toUpperCase() + name.substring(1);
    String getMethod = "get" + fieldName;
    Method method = clazz.getMethod(getMethod);

    Object value = method.invoke(instance);

}  catch (Exception ex) {
     // Handle exceptions
}
```

**Get methods of a type**

```
Class<Foo> clazz = Foo.class;
Method[] methods =  clazz.getMethods();
```

**Get method by name**

```
 try {
    String name = "printBar";
    Class<Foo> clazz = Foo.class;
    Method method = clazz.getDeclaredMethod(name);

} catch (Exception ex) {
    // Handle exceptions
}
```

**Call method by name**

```
try {
    String name = "printBar";
    String param = "Foo";
    Class<Foo> clazz = Foo.class;
    Foo instance = clazz.getDeclaredConstructor().newInstance();
    
    Method method = clazz.getMethod(name, String.class);

    Object value = method.invoke(instance, param);

  
} catch (Exception ex) {
   // Handle exceptions
}
```
