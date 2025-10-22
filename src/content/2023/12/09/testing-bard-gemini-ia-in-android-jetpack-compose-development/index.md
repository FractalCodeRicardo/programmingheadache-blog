---
title: "Testing Bard - Gemini IA in Android Jetpack Compose Development"
date: 2023-12-09
categories: 
  - "bard"
  - "gemini"
  - "jetpack-compose"
image: "/images/testing-image.png"
---

Finally google is competing in the IA world and Gemini IA has been released. You can testing it using Bard:

[https://bard.google.com/](https://bard.google.com/)

Let's see how can we improve our Jetpack programming using it.

## **Test 1. Simple user screen with some fields**

Prompt:

![](/images/testing-image.png)

Output:

```kotlin
@Composable
fun UserScreen() {
    Column(
        modifier = Modifier.padding(16.dp),
        verticalArrangement = Arrangement.spacedBy(8.dp)
    ) {
        Text(
            text = "User Information",
            style = MaterialTheme.typography.titleMedium,
            color = MaterialTheme.colorScheme.primary
        )

        OutlinedTextField(
            value = "",
            onValueChange = {},
            label = { Text("Name") },
            colors = TextFieldDefaults.outlinedTextFieldColors(
                textColor = MaterialTheme.colorScheme.onSurface,
                cursorColor = MaterialTheme.colorScheme.primary,
                focusedBorderColor = MaterialTheme.colorScheme.primary,
                unfocusedBorderColor = MaterialTheme.colorScheme.outline
            ),
            modifier = Modifier.fillMaxWidth()
        )

        OutlinedTextField(
            value = "",
            onValueChange = {},
            label = { Text("Email") },
            colors = TextFieldDefaults.outlinedTextFieldColors(
                textColor = MaterialTheme.colorScheme.onSurface,
                cursorColor = MaterialTheme.colorScheme.primary,
                focusedBorderColor = MaterialTheme.colorScheme.primary,
                unfocusedBorderColor = MaterialTheme.colorScheme.outline
            ),
            modifier = Modifier.fillMaxWidth(),
            keyboardType = KeyboardType.Email
        )

        OutlinedTextField(
            value = "",
            onValueChange = {},
            label = { Text("Phone number") },
            colors = TextFieldDefaults.outlinedTextFieldColors(
                textColor = MaterialTheme.colorScheme.onSurface,
                cursorColor = MaterialTheme.colorScheme.primary,
                focusedBorderColor = MaterialTheme.colorScheme.primary,
                unfocusedBorderColor = MaterialTheme.colorScheme.outline
            ),
            modifier = Modifier.fillMaxWidth(),
            keyboardType = KeyboardType.Phone
        )

        Row(
            modifier = Modifier.fillMaxWidth(),
            horizontalArrangement = Arrangement.SpaceBetween
        ) {
            Text(
                text = "User Type",
                style = MaterialTheme.typography.bodyMedium,
                color = MaterialTheme.colorScheme.onSurface
            )

            RadioGroup(
                selected = 0,
                onSelectedChange = { /*TODO: Implement user type selection*/ },
                colors = RadioGroupDefaults.colors(
                    selectedColor = MaterialTheme.colorScheme.primary,
                    unselectedColor = MaterialTheme.colorScheme.onSurfaceVariant
                )
            ) {
                RadioButton(
                    selected = false,
                    onClick = { /*TODO: Implement Admin selection*/ }
                ) {
                    Text("Admin")
                }
                RadioButton(
                    selected = true,
                    onClick = { /*TODO: Implement Customer selection*/ }
                ) {
                    Text("Customer")
                }
            }
        }

        Button(
            onClick = {},
            modifier = Modifier.align(Alignment.CenterHorizontally),
            colors = ButtonDefaults.buttonColors(
                backgroundColor = MaterialTheme.colorScheme.primary,
                contentColor = MaterialTheme.colorScheme.onPrimary
            )
        ) {
            Text("Submit")
        }
    }
}
```

**Impressions**:

- I had to ask for recent code using material3. If you dont do it, Gemini will give you old jetpack code

- Gemini does not give you the import section part, so you have to ask for it

- It gave me some references that didn't exists

![](/images/testing-image-1.png)

- Something great is that it give you the github references!!

![](/images/testing-image-2.png)

I had to make some changes and this is how it looks like

![](/images/testing-image-3.png)

## **Test 2. Simple field validation**

Im going to ask Gemini for two simple functions:

- A function that validates an email string

- A function that validates a phone number

![](/images/testing-image-4.png)

This is the code

```kotlin
fun isValidEmail(email: String): Boolean {
    val regex = Regex("^([a-zA-Z0-9_\\.\\-\\+]+)@([a-zA-Z0-9\\-]+)\\.([a-zA-Z]{2,6})(\\.[a-zA-Z]{2,6})?$")
    return regex.matches(email)
}

fun isValidPhoneNumber(phoneNumber: String): Boolean {
    val regex = Regex("^\\+\\d{2}-\\d{3}-\\d{7}$")
    return regex.matches(phoneNumber)
}
```

## **Test 3. Simple unit tests**

To test the code it gave me before I'm going to ask for the unit tests of the two functions

![](/images/testing-image-5.png)

This is the code

```kotlin
@Test
fun testValidEmail() {
    val validEmails = listOf(
        "email@gmail.com",
        "email_1@gmail.com",
        "email.1@gmail.com.mx"
    )
    for (email in validEmails) {
        assertTrue(isValidEmail(email))
    }
}

@Test
fun testInvalidEmail() {
    val invalidEmails = listOf(
        "not_an_email",
        "email@invaliddomain",
        "email@.com",
        "email@example@com"
    )
    for (email in invalidEmails) {
        assertFalse(isValidEmail(email))
    }
}

@Test
fun testValidPhoneNumber() {
    val validPhoneNumbers = listOf(
        "+1-234-5678900",
        "+44-789-0123456",
        "+82-10-98765432"
    )
    for (phoneNumber in validPhoneNumbers) {
        assertTrue(isValidPhoneNumber(phoneNumber))
    }
}

@Test
fun testInvalidPhoneNumber() {
    val invalidPhoneNumbers = listOf(
        "not_a_phone_number",
        "1234567890",
        "+1-234",
        "+1234-567890",
        "+1-234-abc-5678"
    )
    for (phoneNumber in invalidPhoneNumbers) {
        assertFalse(isValidPhoneNumber(phoneNumber))
    }
}
```

I ran the test and one of them failed

![](/images/testing-image-6.png)

The regex to validate the phone number needed to be fixed and one of the unit test had an error too.

**Impressions**

In this part most of the code were good but you have to test it well. The fixes that I made were easy, the important part is test code well if you want to use IA as your code partner.

## Test 4. Game question programmer interview

Let's see if Gemini can code the snake game in jetpack!

![](/images/testing-image-7.png)

```kotlin
@Composable
fun SnakeGameScreen(modifier: Modifier = Modifier) {
    Box(modifier = modifier.fillMaxSize()) {
        Canvas(modifier = Modifier.fillMaxSize()) {
            // Draw game area background
            drawRect(color = Color.LightGray)
            // Draw snake body
            for (segment in snakeBody) {
                drawRect(color = Color.Green, size = segmentSize)
            }
            // Draw food
            drawCircle(color = Color.Red, radius = foodSize)
        }
    }
}

// Game state variables
private val snakeBody = mutableStateListOf<Offset>()
private val foodPosition = Offset.Zero
private var direction = Offset(1f, 0f)
private val snakeSpeed = 5f

// Update game state
fun updateGameState() {
    // Update snake head position
    val newHeadPosition = snakeBody.first() + direction * snakeSpeed
    snakeBody.add(0, newHeadPosition)
    // Check for collision with food
    if (newHeadPosition == foodPosition) {
        // Increase snake length
        snakeBody.add(snakeBody.last())
        // Generate new food position
        generateFoodPosition()
    }
    // Check for collision with boundaries or itself
    if (newHeadPosition !in 0f..screenWidth || newHeadPosition in snakeBody.drop(1)) {
        // Game over
    }
    // Remove last segment of the snake
    snakeBody.removeLast()
}

// Generate new food position
private fun generateFoodPosition() {
    foodPosition = Offset(
        x = Random().nextInt(screenWidth).toFloat(),
        y = Random().nextInt(screenHeight).toFloat()
    )
}
```

**Impressions**

In this case the code didn't work at all and I asked a lot of things but it does not work very well. This is the best i can get.

```kotlin

import androidx.compose.foundation.Canvas
import androidx.compose.foundation.layout.Box
import androidx.compose.foundation.layout.fillMaxSize

import androidx.compose.runtime.Composable
import androidx.compose.runtime.mutableStateListOf
import androidx.compose.runtime.remember
import androidx.compose.ui.Modifier
import androidx.compose.ui.geometry.Offset
import androidx.compose.ui.geometry.Size
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.unit.dp
import kotlin.random.Random
import androidx.compose.foundation.ExperimentalFoundationApi
import androidx.compose.foundation.gestures.detectDragGestures
import androidx.compose.ui.ExperimentalComposeUiApi
import androidx.compose.ui.input.pointer.PointerInputChange
import androidx.compose.ui.input.pointer.pointerInput

private val snakeBody = mutableStateListOf<Offset>(Offset(100F, 100F))
private var foodPosition = Offset.Zero
private var direction = Offset(10F,0f)
private val snakeSpeed = 1f

@Composable
fun SnakeGameScreen() {
    Box(modifier = Modifier.fillMaxSize()) {
        Canvas(modifier = Modifier.fillMaxSize().pointerInput(Unit) {
            detectDragGestures { change, dragAmount ->
                if (change.pressed) {
                    val deltaX = dragAmount.x
                    val deltaY = dragAmount.y
                    if (Math.abs(deltaX) > Math.abs(deltaY)) {
                        if (deltaX > 0) {
                            // Swipe right
                            updateGameState(6)
                        } else {
                            // Swipe left
                            updateGameState(4)
                        }
                    } else if (Math.abs(deltaY) > Math.abs(deltaX)) {
                        if (deltaY > 0) {
                            // Swipe down
                            updateGameState(8)
                        } else {
                            // Swipe up
                            updateGameState(2)
                        }
                    }
                }
            }
        }) {
            // Draw game area background
           // drawRect(color = Color.LightGray)
            // Draw snake body
            for (segment in snakeBody) {
               drawCircle(color = Color.Green, radius = 50F,center = segment)
            }
            // Draw food
            drawCircle(color = Color.Red, radius = 50F)
        }
    }
}

// Game state variables

// Update game state
fun updateGameState(key: Int) {
    when (key) {
        4 -> direction = Offset(-10f, 0f)
        6 -> direction = Offset(10f, 0f)
        2 -> direction = Offset(0f, -10f)
        8 -> direction = Offset(0f, 10f)
    }

    // Update snake head position
    val newHeadPosition = snakeBody.first() + direction * snakeSpeed
    snakeBody.add(0, newHeadPosition)
    // Check for collision with food
    if (newHeadPosition == foodPosition) {
        // Increase snake length
        snakeBody.add(snakeBody.last())
        // Generate new food position
        generateFoodPosition()
    }
    // Check for collision with boundaries or itself
    //if (newHeadPosition !in 0f..100F || newHeadPosition in snakeBody.drop(1)) {
        // Game over
    //}
    // Remove last segment of the snake
    snakeBody.removeLast()
}

// Generate new food position
private fun generateFoodPosition() {
    foodPosition = Offset(
        x = Random.nextInt(500).toFloat(),
        y = Random.nextInt(500).toFloat()
    )
}
```

![](/images/Peek-2023-12-09-11-18.gif)

  
**Final thoughts**

- You don't have to write "easy" code anymore, the IA can do it very well.

- Don't trust on every code the IA gives you, most of the time you have to test, fix and analyze the code.

- The IA works well if you asks for small and well defined tasks,

- The IA will fail if the task is big and ambiguous
