---
title: "Effective Singleton Instance Management in Jetpack Compose Using Map"
date: 2024-01-11
categories: 
  - "kotlin"
  - "refactoring"
tags: 
  - "kotlin"
  - "singleton"
  - "view-model"
---

The Singleton design pattern ensures that a class has only one instance and provides a global point of access to it. This pattern is used frecuently in Jetpack Compose and has these benefits:

1. You ensure only one instance of each class, so you have memory optimization.

3. Jetpack Compose screens recompose constantly, having only one instance of each class ensures consistency.

5. Global access, the object creation is simplified having a common path to access the objects.

The pattern is often used in the creation of view models or repositories. Let's see an example and drawbacks of using it over view models.

Suppose we have three entities in our project

```kotlin
data class User(val id: String)
data class Customer(val id: String)
data class Provider(val id: String)
```

And we have a view model for each one:

```kotlin
// Users
interface UsersViewModel {
    fun showUsers()
}

class UsersViewModelImp: UsersViewModel {
    override fun showUsers() {}
}

// Customers
interface CustomersViewModel {
    fun showCostumers()
}

class CustomersViewModelImp: CustomersViewModel {
    override fun showCostumers() {}
}

// Providers
interface ProvidersViewModel {
    fun showProviders()
}

class ProvidersViewModelImp: ProvidersViewModel  {
    override fun showProviders() {}
}
```

Now we want an instance of each view model. An approach could be this one:

```kotlin
object ViewModelFactory {
    val users: UsersViewModel = UsersViewModelImp()
    val customers: CustomersViewModel = CustomersViewModelImp()
    val providers: ProvidersViewModel = ProvidersViewModelImp()
}
```

The problem with this approach is that **you initialize all the instances and this could cause performance issues**. A nice approach is to save our instances in a Map, so we can perform this :  

1. We add the instance in the map only when is necesary.

3. If the instance is already in the map we can use it

5. If the instance is not in the map we have to create and add it.

```kotlin
fun <T : Any> getInstace(key: String, create: () -> T): T {
    // If the instance is not in the map we have to create and add it.
    if (!mapInstances.containsKey(key)) {
        mapInstances[key] = create()
    }

    return mapInstances[key]
}
      
```

Now we can add each singleton

```kotlin
fun users(): UsersViewModel {
    return getInstance("users") { UsersViewModelImp() }
}

fun customers(): CustomersViewModel {
    return getInstance("customers") { CustomersViewModelImp() }
}

fun providers(): ProvidersViewModel {
    return getInstance("providers") { ProvidersViewModelImp() }
}
```

This is the whole code:

```kotlin
class MapViewModelFactory {
    companion object {
        private val mapInstances = mutableMapOf<String, Any>()

        fun <T : Any> getInstance(key: String, create: () -> T): T {
            if (!mapInstances.containsKey(key)) {
                mapInstances[key] = create()
            }
            return mapInstances[key] as T
        }

        fun users(): UsersViewModel {
            return getInstance("users") { UsersViewModelImp() }
        }

        fun customers(): CustomersViewModel {
            return getInstance("customers") { CustomersViewModelImp() }
        }

        fun providers(): ProvidersViewModel {
            return getInstance("providers") { ProvidersViewModelImp() }
        }

    }
}
```

And you can use it like this:

```kotlin
val usersViewModel = MapViewModelFactory.users()
usersViewModel.showUsers()

val customersViewModel = MapViewModelFactory.customers()
customersViewModel.showCostumers()

val providersViewModel = MapViewModelFactory.providers()
providersViewModel.showProviders()
```
