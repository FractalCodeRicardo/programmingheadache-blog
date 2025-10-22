---
title: "Cleaner screens using Gemini-Bard and Extract Composable refactor"
date: 2023-12-14
categories: 
  - "bard"
  - "gemini"
  - "jetpack-compose"
tags: 
  - "android"
  - "bard"
  - "gemini"
  - "jetpack-compose"
  - "kotlin"
  - "martin-fowler"
  - "refactoring"
image: "/images/cleaner-image-10.png"
---

![](/images/cleaner-image-10.png)

In my opinion, one of the best refactors is _extract function._ Martin Fowler describe it in his book _Refactoring Improving the Design of Existing Code_

_"You have a code fragment that can be grouped together. Turn the fragment into a method whose name explains the purpose of the method."_

```kotlin
 void printOwing(double amount) {
   printBanner();
   //print details
   System.out.println ("name:" + _name);
   System.out.println ("amount" + amount);
 }
```

With the method extracted looks like this

```kotlin
void printOwing(double amount) {
   printBanner();
   printDetails(amount);
 }
 
void printDetails (double amount) {
   System.out.println ("name:" + _name);
   System.out.println ("amount" + amount);
 }
```

The same principle can be applied to composables in Jetpack Compose. And it can be achieved easily using Gemini Bard IA.

Consider this example. It is a form for personal information.

```kotlin

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun PersonalInfoScreen(
) {
    Surface {
        Column(
            modifier = Modifier.padding(16.dp),
            verticalArrangement = Arrangement.spacedBy(8.dp)
        ) {
            Text(
                text = "Enter your personal information",
                style = MaterialTheme.typography.bodyLarge,
            )

            // Name and Last Name
            Row(
                modifier = Modifier.fillMaxWidth(),
                horizontalArrangement = Arrangement.SpaceBetween,
            ) {
                OutlinedTextField(
                    value ="",
                    onValueChange = {  },
                    label = { Text("Name") },
                    modifier = Modifier.weight(1f),
                    singleLine = true,
                )
                OutlinedTextField(
                    value = "",
                    onValueChange = {  },
                    label = { Text("Last Name") },
                    modifier = Modifier.weight(1f),
                    singleLine = true,
                )
            }

            // Email
            OutlinedTextField(
                value = "",
                onValueChange = {  },
                label = { Text("Email") },
                modifier = Modifier.fillMaxWidth(),
                singleLine = true,
            )

            // Nationality
            OutlinedTextField(
                value = "",
                onValueChange = {  },
                label = { Text("Nationality") },
                modifier = Modifier.fillMaxWidth(),
                singleLine = true,
            )

            // Birthdate
            DatePicker(
                state = DatePickerState(null, null, IntRange(1900, 2030), DisplayMode.Input) ,
                title = { Text("Birthdate") },
                modifier = Modifier.fillMaxWidth(),
            )

            Button(
                onClick = {},
                modifier = Modifier.fillMaxWidth(),
            ) {
                Text("Submit")
            }
        }
    }
}
```

It would be great if we extract each field into a Composable. This has these benefits:

- You can add a preview for each field

- You can test it individually

- You can make modifications easier

So, lets ask Bard to extract those composables

![](/images/image-11.png)

Here is the code that Bard gives me. I had to fix some errors:

  

```kotlin

@Composable
fun NameField(
    modifier: Modifier = Modifier,
    value: String = "",
    onValueChange: (String) -> Unit,
) {
    OutlinedTextField(
        value = value,
        onValueChange = onValueChange,
        label = { Text("Name") },
        modifier = modifier.fillMaxWidth(),
        singleLine = true,
    )
}

@Composable
fun LastNameField(
    modifier: Modifier = Modifier,
    value: String = "",
    onValueChange: (String) -> Unit,
) {
    OutlinedTextField(
        value = value,
        onValueChange = onValueChange,
        label = { Text("Last Name") },
        modifier = modifier.fillMaxWidth(),
        singleLine = true,
    )
}

@Composable
fun EmailField(
    modifier: Modifier = Modifier,
    value: String = "",
    onValueChange: (String) -> Unit,
) {
    OutlinedTextField(
        value = value,
        onValueChange = onValueChange,
        label = { Text("Email") },
        modifier = modifier.fillMaxWidth(),
        singleLine = true,
    )
}

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun BirthdateField(
    modifier: Modifier = Modifier,
    birthDate: LocalDate? = null,
    onBirthDateChange: (LocalDate?) -> Unit,
) {
    DatePicker(
        state = DatePickerState(null, null, IntRange(1900, 2030), DisplayMode.Input) ,
        title = { Text("Birthdate") },
        modifier = modifier.fillMaxWidth()

    )
}

@Composable
fun SubmitButton(
    modifier: Modifier = Modifier,
    onClick: () -> Unit,
) {
    Button(
        onClick = onClick,
        modifier = modifier.fillMaxWidth(),
    ) {
        Text("Submit")
    }
}

@Composable
fun NationalityField(
    modifier: Modifier = Modifier,
    value: String = "",
    onValueChange: (String) -> Unit,
) {
    OutlinedTextField(
        value = value,
        onValueChange = onValueChange,
        label = { Text("Nationality") },
        modifier = modifier.fillMaxWidth(),
        singleLine = true,
    )
}
```

Now I asked for the original code using this new functions

![](/images/image-12.png)

```kotlin
@Composable
fun PersonalInfoScreen(
    name: String = "",
    onNameChange: (String) -> Unit,
    lastName: String = "",
    onLastNameChange: (String) -> Unit,
    email: String = "",
    onEmailChange: (String) -> Unit,
    nationality: String = "",
    onNationalityChange: (String) -> Unit,
    birthDate: LocalDate? = null,
    onBirthDateChange: (LocalDate?) -> Unit,
    onSubmit: () -> Unit,
) {
    Surface {
        Column(
            modifier = Modifier.padding(16.dp),
            verticalArrangement = Arrangement.spacedBy(8.dp)
        ) {
            Text(
                text = "Enter your personal information",
                style = MaterialTheme.typography.bodyLarge,
            )

            NameField(
                value = name,
                onValueChange = onNameChange,
            )

            LastNameField(
                value = lastName,
                onValueChange = onLastNameChange,
            )

            EmailField(
                value = email,
                onValueChange = onEmailChange,
            )

            NationalityField(
                value = nationality,
                onValueChange = onNationalityChange,
            )

            BirthdateField(
                birthDate = birthDate,
                onBirthDateChange = onBirthDateChange,
            )

            SubmitButton(
                onClick = onSubmit,
            )
        }
    }
}
```

It looks much cleaner!

I ask for the preview and Bard give me this code

```kotlin
@Preview(showBackground = true)
@Composable
fun PersonalInfoScreenPreview() {
    PersonalInfoScreen(
        name = "Jane Doe",
        lastName = "Johnson",
        email = "janedoe@example.com",
        nationality = "Canada",
        birthDate = null,
        onNameChange = { /* Handle name change here */ },
        onLastNameChange = { /* Handle last name change here */ },
        onEmailChange = { /* Handle email change here */ },
        onNationalityChange = { /* Handle nationality change here */ },
        onBirthDateChange = { /* Handle birth date change here */ },
        onSubmit = { /* Handle submit action here */ },
    )
}
```

This is how it looks

![](/images/Peek-2023-12-14-11-34.gif)
