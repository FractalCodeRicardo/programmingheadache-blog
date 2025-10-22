---
title: "Think Once, Reuse Forever: Leveraging Generic Jetpack Composables to Eliminate Duplicate Code"
date: 2024-03-01
categories: 
  - "jetpack-compose"
  - "kotlin"
  - "refactoring"
image: "/images/think-image-4.png"
---

![](/images/think-image-4.png)

The use of generic programming in Jetpack Compose promotes the creation of highly reusable designs and, furthermore, avoids one of the less desired code smells: duplicate code.

We will see a brief and simple example of how to avoid duplicate code using generics.

Consider the following models in our project

```kotlin
data class Beer(
    val name: String = "",
    val alcoholByVolume: Double = 0.0,
    val price: Double = 0.0
)

data class Coffee(
    val name: String = "",
    val price: Double = 0.0,
    val extras: String = ""
)

data class Soda(
    val name: String = "",
    val price: Double = 0.0,
    val sugarGrams: Double = 0.0
)
```

We want to display a list of each model. A naive approach would be create a list of each model. Something like this.

```kotlin
// BEERS
@Composable
fun ListBeers(
    items: List<Beer>,
) {
    LazyColumn(
        modifier = Modifier.fillMaxSize()
    ) {
        this.items(
            items = items,
            itemContent = {
                ElevatedCard(modifier = Modifier.padding(10.dp)) {
                    Column(
                        modifier = Modifier.padding(10.dp)
                    ) {
                        Text(
                            text = it.name,
                            fontWeight = FontWeight.Bold,
                            color = MaterialTheme.colorScheme.primary
                        )

                        Divider()

                        Column (
                            modifier = Modifier.padding(start = 20.dp)
                        ){
                            Text("Name: " + it.name)
                            Text("Alcohol: " + it.alcoholByVolume)
                            Text("Price " + it.price)
                        }
                    }
                }
                Spacer(modifier = Modifier.height(5.dp))
            }
        )
    }
}

// COFFEES
@Composable
fun ListCoffees(
    items: List<Coffee>,
) {
    LazyColumn(
        modifier = Modifier.fillMaxSize()
    ) {
        this.items(
            items = items,
            itemContent = {
                ElevatedCard(
                    modifier = Modifier.padding(10.dp),
                ) {
                    Column(
                        modifier = Modifier.padding(10.dp)
                    ) {
                        Text(
                            text = it.name,
                            fontWeight = FontWeight.Bold,
                            color = MaterialTheme.colorScheme.primary
                        )

                        Divider()

                        Column (
                            modifier = Modifier.padding(start = 20.dp)
                        ){
                            Text("Name: " + it.name)
                            Text("Price: " + it.price)
                            Text("Extras: " + it.extras)
                        }

                    }
                }
                Spacer(modifier = Modifier.height(5.dp))
            }
        )
    }
}

// SODAS
@Composable
fun ListSodas(
    items: List<Soda>,
) {
    LazyColumn(
        modifier = Modifier.fillMaxSize()
    ) {
        this.items(
            items = items,
            itemContent = {
                ElevatedCard(modifier = Modifier.padding(10.dp)) {
                    Column(
                        modifier = Modifier.padding(10.dp)
                    ) {
                        Text(
                            text = it.name,
                            fontWeight = FontWeight.Bold,
                            color = MaterialTheme.colorScheme.primary
                        )

                        Divider()

                        Column (
                            modifier = Modifier.padding(start = 20.dp)
                        ){
                            Text("Name: " + it.name)
                            Text("Price: " + it.price)
                            Text("Sugar Grams: " + it.sugarGrams)
                        }

                    }
                }
                Spacer(modifier = Modifier.height(5.dp))
            }
        )
    }
}
```

The lists look like this:

![](/images/image.png)

![](/images/think-image-1.png)

![](/images/think-image-2.png)

As you can see, the lists are very similar; they share the same structure, title, and content details. Therefore, we can remove a lot of duplicate code by creating a generic list.

We will send the common elements (title and content details) as parameters, and we will use generics to specify the type.

```kotlin
@Composable
fun <T: Any> GenericList(
    items: List<T>,
    title: String,
    contentDetails: @Composable (T) -> Unit,
) {

}
```

And we use the title and content

```kotlin
@Composable
fun <T: Any> GenericList(
    items: List<T>,
    title: String,
    contentDetails: @Composable (T) -> Unit,
) {
    LazyColumn(
        modifier = Modifier.fillMaxSize()
    ) {
        this.items(
            items = items,
            itemContent = {
                ElevatedCard(modifier = Modifier.padding(10.dp)) {
                    Column(
                        modifier = Modifier.padding(10.dp)
                    ) {
                        Text(
                            text = title, // Title
                            fontWeight = FontWeight.Bold,
                            color = MaterialTheme.colorScheme.primary
                        )

                        Divider()

                        Column (
                            modifier = Modifier.padding(start = 20.dp)
                        ){
                            contentDetails(it) // Content Details
                        }

                    }
                }
                Spacer(modifier = Modifier.height(5.dp))
            }
        )
    }
}
```

Now, see how clean the new lists are

```kotlin
// BEERS
@Composable
fun NewBeersList(items: List<Beer>) {
    GenericList( items,"Beers",
        contentDetails = {
            Text("Name: " + it.name)
            Text("Alcohol: " + it.alcoholByVolume)
            Text("Price " + it.price)
        }
    )
}

// COFFEES
@Composable
fun NewCoffeesList(items: List<Coffee>) {
    GenericList(items,"Coffees",
        contentDetails = {
            Text("Name: " + it.name)
            Text("Price: " + it.price)
            Text("Extras: " + it.extras)
        }
    )
}

// SODAS
@Composable
fun NewSodasList(items: List<Soda>) {
    GenericList(items,"Sodas",
        contentDetails = {
            Text("Name: " + it.name)
            Text("Price: " + it.price)
            Text("Sugar Grams: " + it.sugarGrams)
        }
    )
}
```

- Github: [https://github.com/FractalCodeRicardo](https://github.com/FractalCodeRicardo)

- Medium: [https://medium.com/@nosilverbullet](https://medium.com/@nosilverbullet )

- Web page: [https://programmingheadache.com](https://programmingheadache.com )

- Source Code: [https://github.com/FractalCodeRicardo/programmingheadache-misc/blob/main/generics-composable-duplicate-code/app/src/main/java/com/thisisthetime/genericscomposableduplicatecode/MainActivity.kt]( https://github.com/FractalCodeRicardo/programmingheadache-misc/blob/main/generics-composable-duplicate-code/app/src/main/java/com/thisisthetime/genericscomposableduplicatecode/MainActivity.kt)
