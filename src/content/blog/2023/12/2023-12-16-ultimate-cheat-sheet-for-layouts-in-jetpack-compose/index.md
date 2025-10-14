---
title: "Ultimate cheat sheet for Layouts in Jetpack Compose"
date: 2023-12-16
categories: 
  - "jetpack-compose"
  - "kotlin"
---

The basics for controlling how elements are showing in Compose consists in the use of _**Row**_ and _**Column**_. Here is the main idea of both:

- _**Row** will show the elements in a single row from left to right._

- _**Column** will show the elements in a single column from top to bottom._

Lets see the common cases. We will use this composable for the examples:

```
@Composable
fun Element(text: String) {
   Column(modifier = Modifier.background(MaterialTheme.colorScheme.primary)
       .border(border = BorderStroke(2.dp, MaterialTheme.colorScheme.onPrimary))
       .width(50.dp)
       .height(50.dp),
       verticalArrangement = Arrangement.Center,
       horizontalAlignment = Alignment.CenterHorizontally) {
       Text(
           text = text,
           color = MaterialTheme.colorScheme.onPrimary,
           fontWeight = FontWeight.Bold
       )
   }
}
```

**One row x Nth elements**

![](images/image-15.png)

```
Row() {
    Element(text = "1")
    Element(text = "2")
    Element(text = ".")
    Element(text = ".")
    Element(text = "N")
}
```

**One column x Nth elements**

![](images/image-16.png)

```
Column() {
    Element(text = "1")
    Element(text = "2")
    Element(text = ".")
    Element(text = ".")
    Element(text = "N")
}
```

**Nth Elements x Nth Elements**

![](images/image-17.png)

```
Column() {
    Row() {
        Element(text = "1")
        Element(text = "2")
        Element(text = "3")
    }

    Row() {
        Element(text = "1")
        Element(text = "2")
        Element(text = "3")
    }

    Row() {
        Element(text = "1")
        Element(text = "2")
        Element(text = "3")
    }
}
```

**Nth Elements **in a Column** Centered**

![](images/X1vO8VcB7DQAAAABJRU5ErkJggg==)

Vertically

![](images/image-18.png)

Horizontally

![](images/image-19.png)

Full centered

```
Column(
    modifier = Modifier.fillMaxSize(),
    verticalArrangement = Arrangement.Center,
    horizontalAlignment = Alignment.CenterHorizontally
) {
    Element(text = "1")
    Element(text = "2")
    Element(text = ".")
    Element(text = "N")
}
```

**Nth Elements **in a Row** Centered**

![](images/image-20.png)

Vertically

![](images/image-23.png)

Horizontally

![](images/image-22.png)

Full centered

```
Row(
        modifier = Modifier.fillMaxSize(),
        horizontalArrangement = Arrangement.Center,
        verticalAlignment = Alignment.CenterVertically
) {
    Element(text = "1")
    Element(text = "2")
    Element(text = ".")
    Element(text = "N")
}
```

**Other alignments in a Column** **(or Row)**

![](images/image-30.png)

Start

![](images/image-31.png)

Center

![](images/image-32.png)

End

```
Column(
    modifier = Modifier.fillMaxSize(),
    // Uncomment the one you need
    //horizontalAlignment = Alignment.Start,
    //horizontalAlignment = Alignment.CenterHorizontally,
    horizontalAlignment = Alignment.End,
) {
    Element(text = "1")
    Element(text = "2")
}
```

**Other arragements in a Row (or Column)**

![](images/image-28.png)

Space between

![](images/image-27.png)

Space around (space after and before are the same)

![](images/image-29.png)

Space evenly (all spaces are the same)

```
Row(
    modifier = Modifier.fillMaxSize(),
    // Uncomment the one that you needs
    //horizontalArrangement = Arrangement.SpaceBetween,
    //horizontalArrangement = Arrangement.SpaceAround,
    horizontalArrangement = Arrangement.SpaceEvenly,
) {
    Element(text = "1")
    Element(text = "2")
}
```

**Nested rows in columns**

![](images/image-33.png)

```
Column() {
    Column {
        Element("C1")
        Element("C2")
    }

    Row() {
        Element("R1")
        Element("R2")
        Element("R3")
    }
    Row() {
        Element("R1")
        Element("R2")
        Element("R3")
    }

    Column {
        Element("C1")
        Element("C2")
        Element("C3")
    }
}
```

**Nested columns in rows**

![](images/image-35.png)

```
Row() {
    Row {
        Element("R1")
        Element("R2")
    }

    Column() {
        Element("C1")
        Element("C2")
        Element("C3")
    }
    Column() {
        Element("C1")
        Element("C2")
        Element("C3")
    }

    Row {
        Element("R1")
        Element("R2")
    }
}
```

**Modifiers Width and Height**

You can set **height** and **width** with this methods of the Modifier class:

- To set fixed **height** use **_Modifier.height_** method

- To set fixed **width** use **_Modifier.width_** method

- To set **max width** use **_Modifier.fillMaxWidth_**

- To set **max height** use **_Modifier.fillMaxHeight_**

- To set **max width and height** use **_Modifier.fillMaxSize_**

- To set the width/height size **relative to parent** use _**Modifier.fillMaxWidth/Modifier.fillMaxHeight**_ using the **fraction parameter**.

Lets see some examples

**Rows width**

![](images/image-36.png)

```
Column() {

    Row(modifier = Modifier.fillMaxWidth()) {
        ElementFillSize(text = "Max width")
    }

    Row(modifier = Modifier.width(200.dp)) {
        ElementFillSize(text = "200dp")
    }

    Row(modifier = Modifier.fillMaxWidth(0.5f)) {
        ElementFillSize(text = "50%")
    }
}
```

**Columns Height**

![](images/image-37.png)

```
Row() {

    Column(modifier = Modifier.fillMaxHeight()) {
        ElementFillSize(text = "Max width")
    }

    Column(modifier = Modifier.height(200.dp)) {
        ElementFillSize(text = "200dp")
    }

    Column(modifier = Modifier.fillMaxHeight(0.5f)) {
        ElementFillSize(text = "50%")
    }
}
```
