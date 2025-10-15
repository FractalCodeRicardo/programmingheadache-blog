---
title: "Kotlin - Reified type parameters in generic classes"
date: 2023-11-07
categories: 
  - "kotlin"
---

Reflection is great, it allows programmers examine and modify objects and classes at runtime. Klotin has a special feature called reified types that facilitates the use of generics.

Consider this

```kotlin
fun <T: Any> printNameClass(clazz: KClass<T>) {
    println(clazz.simpleName)
}

// Output: String
fun main() {
    printNameClass(String::class)
}
```

You can remove the clazz parameter using reified types and inline functions like this

```kotlin
inline fun <reified T> printNameClass() {
    println(T::class.simpleName)
}

// Output: String
fun main() {
    printNameClass<String>()
}
`

This is fine cause you don't need to pass the clazz parameter any more. But what about generic classes?

First option, using the generics without reified kerword. This brings to an error _Cannot use 'T' as reified type parameter. Use a class instead_

```kotlin
class GenericClass<T: Any> {
    fun printName() {
        println(T::class) // Error:Cannot use 'T' as reified type parameter. Use a class instead.
    }
}
```

Second option, adding reified keyword brings the error _Only type parameters of inline functions can be reified_

```kotlin
class GenericClass<reified T: Any> { //Error: Only type parameters of inline functions can be reified
    fun printName() {
        println(T::class)
    }
}
```

So, when using generic classes you have to pass de KClass parameter

```kotlin
class GenericClass<T: Any>(
    private val clazz: KClass<T>
) {
    fun printName() {
        println(clazz.simpleName)
    }
}

// Output: String
fun main() {
    var genericClass = GenericClass(String::class)
    genericClass.printName()
}
```
