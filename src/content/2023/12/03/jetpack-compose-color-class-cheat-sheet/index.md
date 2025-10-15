---
title: "Jetpack Compose Color class Cheat Sheet"
date: 2023-12-03
categories: 
  - "jetpack-compose"
  - "kotlin"
---

Here are some useful codes for handling _androidx.compose.ui.graphics.Color_ class in Jetpack Compose

Predefined colors that you can use

```kotlin
var red = Color.Red
var black = Color.Black
var blue = Color.Blue
var cyan = Color.Cyan
var darkGray = Color.DarkGray
var gray = Color.Gray
var green = Color.Green
var lightGray = Color.LightGray
var magenta = Color.Magenta
var white = Color.White
var yellow = Color.Yellow
```

Create a color from RGB

```kotlin
// Create color from RGB
val red = 255
val green = 255
val blue = 255

val color = Color(red, green, blue)
```

Create a color from RGBA

```kotlin
// Create color from RGBA
val red = 255
val green = 255
val blue = 255
val alpha = 255 // 0: transparent, 255: solid

val color = Color(red, green, blue, alpha)
```

Create a color from hexadecimal RGB string

```kotlin
// Create from hexadecimal
val hexadecimal = "#AABBCC" // AA: Green, BB: Red, CC: Blue
val color  = Color(hexadecimal.toColorInt())
```

Create a color from hexadecimal RGBA string

```kotlin
// Create from hexadecimal
// FF: Alpha (00 transparent FF Solid) AA: Green, BB: Red, CC: Blue
val hexadecimal = "#FFAABBCC"
val color  = Color(hexadecimal.toColorInt())
```

Create color from name

```kotlin
val red = "red"
val color = Color(red.toColorInt())

/* Valid strings: red, blue, green, black, white,
    gray, cyan, magenta, yellow, lightgray, darkgray,
    grey, lightgrey, darkgrey, aqua, fuchsia, lime,
    maroon, navy, olive, purple, silver, teal. */
```

Create color from hexadecimal number

```kotlin
// FF: Alpha (00 transparent FF Solid) AA: Green, BB: Red, CC: Blue
val hex = 0xFFAABBCC
val color = Color(hex)
```

Represent color as an integer

```kotlin
val red = Color.Red
val intColor = red.toArgb()

// You can create the color with intColor
val redColor = Color(intColor)
```

Represent color as an String

```kotlin
val red = Color.Red

// #FF0000
val hex = "#" + Integer.toHexString(red.toArgb()).uppercase()
// You can create the color with hex
val redColor = Color(hex.toColorInt())
```
