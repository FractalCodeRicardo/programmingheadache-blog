---
title: "Building a Generic Searchable Table Dialog in Jetpack Compose"
date: 2023-12-21
categories: 
  - "jetpack-compose"
  - "kotlin"
image: "/images/Peek-2023-12-20-17-37-4.gif"
---

![](/images/Peek-2023-12-20-17-37-4.gif)

In certain scenarios, users need to search and select a record from a table. Implementing a dialog that displays records for user selection would be great. The goal is to create a versatile dialog that is not tied to a specific record type, enabling easy reuse across different contexts.

## **Creating data**

First, lets create a data class and a list for the example

```kotlin
data class Animal(
    val name: String = "",
    val type: String = "",
    val height: Double = 0.0,
    val weight: Double = 0.0
)

val animals = listOf(
    Animal(name = "Lion", type = "Mammal", height = 3.5, weight = 200.0),
    Animal(name = "Eagle", type = "Bird", height = 0.8, weight = 5.0),
    Animal(name = "Fish", type = "Aquatic", height = 0.2, weight = 0.5),
    Animal(name = "Elephant", type = "Mammal", height = 9.0, weight = 5000.0),
    Animal(name = "Snake", type = "Reptile", height = 1.5, weight = 10.0),
    Animal(name = "Dolphin", type = "Aquatic", height = 2.0, weight = 300.0)
)
```

We want to know the name and the order of the columns. We are going to use an annotation to do it.

```kotlin
@Retention(AnnotationRetention.RUNTIME)
annotation class ColumnTable(
    val name: String,
    val order: Int = 0
)
```

And we use it over the animals class

```kotlin
data class Animal(

    @property: ColumnTable("Name", 0)
    val name: String = "",

    @property: ColumnTable("Type", 1)
    val type: String = "",

    @property: ColumnTable("Height", 2)
    val height: Double = 0.0,

    @property: ColumnTable("Weight", 3)
    val weight: Double = 0.0
)
```

## Creating Headers

Lets start to build the table. First, we want to show the column headers. We need two things:

1. Iterate over class properties

3. Get the ColumnTable annotation an retrieve tha name of the column

We are going to use generics so you have to add this dependency (update the version)

```kotlin
implementation("org.jetbrains.kotlin:kotlin-reflect:1.9.21")
```

Now you can access to the List of properties with this code

```kotlin
var properties = T::class.memberProperties // Where T is Animals in this case
```

And we can use a function like this to get the ColumnTable annotation

```kotlin
fun <T : Any> findAnnotation(property: KProperty1<T, *>): ColumnTable? {
    return property
        .annotations
        .find { a -> a is ColumnTable } as? ColumnTable
}
```

To get the column names we are going to use this function

```kotlin
inline fun <reified T : Any> columnsNames(): List<String> {
    val properties = T::class.memberProperties

    return properties
        .mapNotNull { findAnnotation(it) }
        .sortedBy { it.order }
        .map { it.name }
}
```

Now lets create a simple composable that show those column names

```kotlin
@Composable()
inline fun <reified T : Any> Headers() {
    val columns = columnsNames<T>()

    Row() {
        columns.forEach {

            Text(text = it)
            Spacer(modifier = Modifier.width(5.dp))
        }
    }
}
```

![](/images/image-43.png)

## Creating Rows

Next, we are going to show the records. To do this we have to retrieve the values of each property of the record. We need to use the method **_get_** for this. For example, if we cant to create a list of the values of a single record we can do it in the following way

```kotlin
 val properties = T::class.memberProperties
 val values = properties.map { it.get(instance).toString() }
```

Lets put this code into a function and we add sorting and filter

```kotlin
inline fun <reified T : Any> values(instance: T): List<String> {
    val properties = T::class.memberProperties

    return properties
        .map {
            object {
                val property = it
                val annotation = findAnnotation(it)
            }
        }
        .filter { it.annotation != null }
        .sortedBy { it.annotation?.order }
        .map { it.property.get(instance).toString() }
}
```

And we create two composables. One for creating each cell and another to create the row containing these cells

```kotlin
@Composable
inline fun <reified T : Any> Cells(instance: T) {
    val values = values(instance)
    Row() {
        values.forEach {

            Text(it)
            Spacer(modifier = Modifier.width(5.dp))
        }

    }
}

@Composable
inline fun <reified T : Any> RowCells(items: List<T>) {
    items.forEach {
        Cells(
            instance = it
        )
    }
}
```

![](/images/image-44.png)

## Adding searchable functionality

Lets add a TextField for the search and put header and rows together.

```kotlin
@OptIn(ExperimentalMaterial3Api::class)
@Composable
inline fun <reified T : Any> SearchableTable(items: List<T>) {
    var text by remember { mutableStateOf("") }
  
    Column {
        TextField(
            value = text,
            onValueChange = {
                text = it
            },
            placeholder = { Text("Type to search") }
        )

        Headers<T>()
        RowCells(filteredItems.value)
    }
}
```

![](/images/Peek-2023-12-20-17-37.gif)

To filter the records we are going to use next two functions. If any of the record values match the search text, then we keep the record.

```kotlin
inline fun <reified T : Any> containsText(text: String, instance: T): Boolean {
    val values = values(instance)
    val lowerText = text.lowercase()
    for (value in values) {
        if (value == null) continue

        if (value.lowercase().contains(lowerText)) {
            return true
        }
    }

    return false
}

inline fun <reified T : Any> filterItems(textSearch: String, items: List<T>): List<T> {
    return items
        .filter { containsText(textSearch, it) }
}
```

And we use it in the composable

```kotlin
@OptIn(ExperimentalMaterial3Api::class)
@Composable
inline fun <reified T : Any> SearchableTable(items: List<T>) {
    var text by remember { mutableStateOf("") }
    val filteredItems = remember { mutableStateOf(items) }

    Column {
        TextField(
            value = text,
            onValueChange = {
                text = it
                filteredItems.value = filterItems(it, items) // filtered items
            },
            placeholder = { Text("Type to search") }
        )

        Headers<T>()
        RowCells(filteredItems.value)
    }
}
```

![](/images/Peek-2023-12-20-17-37-1.gif)

## Creating the dialog

Lets wrap what we have done into a dialog and make some visual improvements

```kotlin
@Composable
inline fun <reified T : Any> SearchableTableDialog(
    title: String,
    items: List<T>,
    onSelected: (T) -> Unit,
    crossinline onDismissRequest: () -> Unit
) {
    Dialog(
        onDismissRequest = { onDismissRequest() },
        properties = DialogProperties(usePlatformDefaultWidth = false),
    ) {

        Surface(
            modifier = Modifier
                .fillMaxWidth(0.9f)
                .fillMaxHeight(0.5f),

            shape = RoundedCornerShape(5.dp),
            border = BorderStroke(1.dp, color = MaterialTheme.colorScheme.primary)
        ) {
            Column(
                modifier = Modifier.padding(10.dp),
                horizontalAlignment = Alignment.CenterHorizontally
            ) {
                Text(
                    modifier = Modifier.fillMaxWidth(),
                    text = title,
                    fontWeight = FontWeight.Bold,
                    fontSize = 20.sp,
                    textAlign = TextAlign.Center,
                    color = MaterialTheme.colorScheme.primary
                )
                SearchableTable(items)
                Spacer(modifier = Modifier.height(10.dp))
                Button(
                    onClick = {},

                    ) {
                    Text("Accept")
                }
            }

        }
    }
}
```

![](/images/Peek-2023-12-20-17-37-2.gif)

Finally we add the events for selecting the record. This is the final code

```kotlin

import androidx.compose.foundation.BorderStroke
import androidx.compose.foundation.background
import androidx.compose.foundation.clickable
import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.Row
import androidx.compose.foundation.layout.Spacer
import androidx.compose.foundation.layout.fillMaxHeight
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.height
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.layout.width
import androidx.compose.foundation.shape.RoundedCornerShape
import androidx.compose.material3.Button
import androidx.compose.material3.ExperimentalMaterial3Api
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Surface
import androidx.compose.material3.Text
import androidx.compose.material3.TextField
import androidx.compose.runtime.Composable
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.runtime.setValue
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.text.style.TextAlign
import androidx.compose.ui.tooling.preview.Preview
import androidx.compose.ui.unit.dp
import androidx.compose.ui.unit.sp
import androidx.compose.ui.window.Dialog
import androidx.compose.ui.window.DialogProperties
import com.thisisthetime.genericsearchlist.ui.theme.AppTheme
import kotlin.reflect.KProperty1
import kotlin.reflect.full.memberProperties

@Retention(AnnotationRetention.RUNTIME)
annotation class ColumnTable(
    val name: String,
    val order: Int = 0,
    val isId: Boolean = false
)

data class Animal(

    @property: ColumnTable("Name", 0, true)
    val name: String = "",

    @property: ColumnTable("Type", 1)
    val type: String = "",

    @property: ColumnTable("Height", 2)
    val height: Double = 0.0,

    @property: ColumnTable("Weight", 3)
    val weight: Double = 0.0
)

val animals = listOf(
    Animal(name = "Lion", type = "Mammal", height = 3.5, weight = 200.0),
    Animal(name = "Eagle", type = "Bird", height = 0.8, weight = 5.0),
    Animal(name = "Fish", type = "Aquatic", height = 0.2, weight = 0.5),
    Animal(name = "Elephant", type = "Mammal", height = 9.0, weight = 5000.0),
    Animal(name = "Snake", type = "Reptile", height = 1.5, weight = 10.0),
    Animal(name = "Dolphin", type = "Aquatic", height = 2.0, weight = 300.0)
)

fun <T : Any> findAnnotation(property: KProperty1<T, *>): ColumnTable? {
    return property
        .annotations
        .find { a -> a is ColumnTable } as? ColumnTable
}

inline fun <reified T : Any> columnsNames(): List<String> {
    val properties = T::class.memberProperties

    return properties
        .mapNotNull { findAnnotation(it) }
        .sortedBy { it.order }
        .map { it.name }
}

inline fun <reified T : Any> values(instance: T): List<String> {
    val properties = T::class.memberProperties

    return properties
        .map {
            object {
                val property = it
                val annotation = findAnnotation(it)
            }
        }
        .filter { it.annotation != null }
        .sortedBy { it.annotation?.order }
        .map { it.property.get(instance).toString() }
}

inline fun <reified T : Any> identifier(instance: T): String {
    val properties = T::class.memberProperties

    val id = properties
        .mapNotNull {
            object {
                val property = it
                val annotation = findAnnotation(it)
            }
        }
        .filter { it.annotation?.isId ?: false }
        .map { it.property }
        .firstOrNull()

    if (id != null) {
        return id.get(instance).toString()
    }

    return ""
}

@Composable()
inline fun <reified T : Any> Headers() {
    val columns = columnsNames<T>()

    Row(
        modifier = Modifier
            .background(MaterialTheme.colorScheme.primary)
            .fillMaxWidth()
    ) {
        columns.forEach {

            Text(
                text = it,
                fontWeight = FontWeight.Bold,
                color = MaterialTheme.colorScheme.onPrimary,
                modifier = Modifier.weight(1f)
            )

        }
    }
}

@Composable
inline fun <reified T : Any> Cells(
    instance: T,
    crossinline onClick: (T) -> Unit,
    selected: Boolean
) {
    val values = values(instance)
    Row(
        modifier = Modifier.clickable {
            onClick(instance)
        }
    ) {
        values.forEach {

            Text(
                text = it,
                fontWeight = if (selected) FontWeight.Bold else FontWeight.Normal,
                color = MaterialTheme.colorScheme.primary,
                modifier = Modifier.weight(1f)
            )
            Spacer(modifier = Modifier.width(5.dp))
        }

    }
}

@Composable
inline fun <reified T : Any> RowCells(
    items: List<T>,
    crossinline onClick: (T) -> Unit
) {
    var id by remember { mutableStateOf("") }
    items.forEach {
        Cells(
            instance = it,
            onClick = {
                id = identifier(it)
                onClick(it)
            },
            selected = id == identifier(it)
        )
    }
}

inline fun <reified T : Any> containsText(text: String, instance: T): Boolean {
    val values = values(instance)
    val lowerText = text.lowercase()
    for (value in values) {
        if (value == null) continue

        if (value.lowercase().contains(lowerText)) {
            return true
        }
    }

    return false
}

inline fun <reified T : Any> filterItems(textSearch: String, items: List<T>): List<T> {
    return items
        .filter { containsText(textSearch, it) }
}

@OptIn(ExperimentalMaterial3Api::class)
@Composable
inline fun <reified T : Any> SearchableTable(
    items: List<T>,
    crossinline onSelected: (T) -> Unit
) {
    var text by remember { mutableStateOf("") }
    val filteredItems = remember { mutableStateOf(items) }

    Column {
        TextField(
            modifier = Modifier.fillMaxWidth(),
            value = text,
            onValueChange = {
                text = it
                filteredItems.value = filterItems(it, items)
            },
            placeholder = { Text("Type to search") }
        )

        Headers<T>()
        RowCells(
            items = filteredItems.value,
            onClick = { onSelected(it) })
    }
}

@Composable
inline fun <reified T : Any> SearchableTableDialog(
    title: String,
    items: List<T>,
    crossinline onSelected: (T?) -> Unit,
    crossinline onDismissRequest: () -> Unit
) {
    var selectedItem by remember { mutableStateOf<T?>(null) }
    Dialog(
        onDismissRequest = { onDismissRequest() },
        properties = DialogProperties(usePlatformDefaultWidth = false),
    ) {

        Surface(
            modifier = Modifier
                .fillMaxWidth(0.9f)
                .fillMaxHeight(0.5f),

            shape = RoundedCornerShape(5.dp),
            border = BorderStroke(1.dp, color = MaterialTheme.colorScheme.primary)
        ) {
            Column(
                modifier = Modifier.padding(10.dp),
                horizontalAlignment = Alignment.CenterHorizontally
            ) {
                Text(
                    modifier = Modifier.fillMaxWidth(),
                    text = title,
                    fontWeight = FontWeight.Bold,
                    fontSize = 20.sp,
                    textAlign = TextAlign.Center,
                    color = MaterialTheme.colorScheme.primary
                )
                SearchableTable(
                    items = items,
                    onSelected = {
                        selectedItem = it
                    })
                Spacer(modifier = Modifier.height(10.dp))
                Button(
                    onClick = {
                        onSelected(selectedItem)
                    },

                    ) {
                    Text("Accept")
                }
            }

        }
    }
}

@Composable
@Preview
fun SearchableTableDialogPreview() {

    var openAnimals by remember { mutableStateOf(false) }
    var selectedAnimal by remember { mutableStateOf("Not selected")}
    AppTheme {
        Surface(

        ) {
            Column(
                modifier = Modifier.fillMaxSize(),
                horizontalAlignment = Alignment.CenterHorizontally,
                verticalArrangement = Arrangement.Center
            ) {
                Button(onClick = { openAnimals = true }) {
                    Text("Select animal")
                }

                Text(text = selectedAnimal)

                if (openAnimals) {
                    SearchableTableDialog(
                        title = "Animals",
                        items = animals,
                        onSelected = {
                            openAnimals = false
                           if (it !=null) {
                               selectedAnimal = it.name
                           }
                        },
                        onDismissRequest = {
                            openAnimals = false
                        }
                    )
                }
            }
        }
    }
}
```

![](/images/Peek-2023-12-20-17-37-3.gif)
