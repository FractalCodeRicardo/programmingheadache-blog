---
title: "Effortless TextField Validation with Custom Annotations in Jetpack Compose: Simplifying TextField Validation"
date: 2024-03-06
categories: 
  - "jetpack-compose"
tags: 
  - "generic-validations"
  - "jetpack-compose"
  - "kotlin"
image: "/images/Peek-2024-03-05-18-03.gif"
---

![](/images/Peek-2024-03-05-18-03.gif)

When building Jetpack Compose applications, one of the most tedious and boring tasks is to validate forms. We want to create code that we want reuse to achieve this task faster. It would be great if we can add some custom validation annotations in our state class to validate its properties.

If you want, you can watch this video where I write the code used in this article.

https://www.youtube.com/watch?v=KrqE1H8l3jc

Suppose we have a user screen like this

![](/images/effortless-text-image-5.png)

We would like to have some annotations in our state that perform the validation. Something like this

```kotlin
data class UserState(
    
    @property:NotEmptyValidation() // <- we want this
    val name: String = "",

    @property:EmailValidation() // <- we want this
    val email: String = "",

    @property:PasswordValidation() // <- we want this
    val password: String = ""
    
)
```

We will use reflection, so ensure you have this line in your build.gradle file (replace the versión if it is needed):

```kotlin
implementation("org.jetbrains.kotlin:kotlin-reflect:1.9.22")
```

1. Firstly, lets create the annotations

```kotlin
@Target( AnnotationTarget.PROPERTY)
annotation class NotEmptyValidation()

@Target( AnnotationTarget.PROPERTY)
annotation class EmailValidation()

@Target( AnnotationTarget.PROPERTY)
annotation class PasswordValidation()
```

2\. Second, lets create a class to store the result of the validation

```kotlin
data class ValidationError(
    val property: String = "",
    val message: String = ""
)
```

3\. We add this class in the state

```kotlin
data class UserState(

    @property:NotEmptyValidation()
    val name: String = "",

    @property:EmailValidation()
    val email: String = "",

    @property:PasswordValidation()
    val password: String = "",

    val error: ValidationError? = null // <- we add this

)
```

4\. Now we are going to create a class to perform the validation. The idea of the class is to iterate over the state properties, find the annotations and perform the validations.

```kotlin
class ValidateState<State: Any>(
    // we are going to use reflection so we pass the KClass instance
    val kClass: KClass<State>
) {
    fun validate(state: State): ValidationError? {
        // TODO: iterate and return first validation error
        return null;
    }
}
```

5\. We iterate over the properties to find the annotations

```kotlin
 fun validate(state: State): ValidationError? {
        kClass.memberProperties.forEach {
            if (it.annotations.isEmpty())
                return@forEach

            val annotation = it.annotations[0];

            if (annotation is NotEmptyValidation) {
                // Perform validation if error found return
            }

            if (annotation is EmailValidation) {
                // Perform validation if error found return
            }

            if (annotation is EmailValidation) {
                // Perform validation if error found return
            }
            
        }

        return null
    }
```

The whole class looks like this

```kotlin
class ValidateState<State: Any>(
    // we are going to use reflection so we pass the KClass instance
    val kClass: KClass<State>
) {
    fun validate(state: State): ValidationError? {
        kClass.memberProperties.forEach {
            if (it.annotations.isEmpty())
                return@forEach

            val annotation = it.annotations[0];
            val property = it.name
            val value = it.get(state)

            if (annotation is NotEmptyValidation) {
                if (isEmpty(value)) {
                    return ValidationError(property, "Must fill this field")
                }
            }

            if (annotation is EmailValidation) {
                if (isNotEmail(value)) {
                    return ValidationError(property, "Not valid email")
                }

            }

            if (annotation is PasswordValidation) {
                 if (isNotPassword(value)) {
                    return ValidationError(property, "Not valid password")
                }
            }

        }

        return null
    }

    private fun isEmpty(value: Any?): Boolean {
        return value.toString().isEmpty()
    }

    private fun isNotEmail(value: Any?): Boolean{
       return !Regex("^[A-Za-z](.*)([@]{1})(.{1,})(\\.)(.{1,})")
           .matches(value.toString())
    }

    private fun isNotPassword(value: Any?): Boolean{
        return !Regex("^(?=.*[a-zA-Z])(?=.*\\d)[a-zA-Z\\d]{5,}$")
            .matches(value.toString());
    }

}
```

6\. Next we use the class in the save method of our view model

```kotlin
    fun save() {
        val stateValidator = ValidateState(UserState::class);
        val error = stateValidator.validate(state.value)

        _state.update {
            it.copy(error = error)
        }
    }
```

7\. In the view, we add the validation message if the error exists. For example, the name field would look like this

```kotlin
 @OptIn(ExperimentalMaterial3Api::class)
@Composable
fun UserScreen(viewModel: UserViewModel) {
    val state by viewModel.state.collectAsState()
    
    // Code ommited
    
     OutlinedTextField(
        label = { Text("Name") },
        value = state.name,
        onValueChange = {
            viewModel.updateProperty(state.copy(name = it))
        },
        isError = state.error?.property == "name" // Here we use the validation error instance
    )
    
    // Code ommited
}
```

8\. Finally we add the error message in a text field

```kotlin

 @OptIn(ExperimentalMaterial3Api::class)
@Composable
fun UserScreen(viewModel: UserViewModel) {
    val state by viewModel.state.collectAsState()
    
    // Code ommited
    
    Text(
        text = state.error?.message?:"",
        color =  MaterialTheme.colorScheme.error
    )
    // Code ommited
}
```

The final result looks like this

![](/images/Peek-2024-03-05-18-03.gif)

**Now you can validate a lot of forms with just few lines of code!**

This is the whole code if you want to see details

```kotlin

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.Spacer
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.height
import androidx.compose.foundation.layout.padding
import androidx.compose.material3.Button
import androidx.compose.material3.ExperimentalMaterial3Api
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.OutlinedCard
import androidx.compose.material3.OutlinedTextField
import androidx.compose.material3.Surface
import androidx.compose.material3.Text

import androidx.compose.runtime.Composable
import androidx.compose.runtime.collectAsState
import androidx.compose.runtime.getValue
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.tooling.preview.Preview
import androidx.compose.ui.unit.dp
import androidx.lifecycle.ViewModel
import com.thisisthetime.customvalidationannotation.ui.theme.AppTheme
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.update
import kotlin.reflect.KClass
import kotlin.reflect.full.memberProperties

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            AppTheme {
                // A surface container using the 'background' color from the theme
                Surface(
                    modifier = Modifier.fillMaxSize(),
                    color = MaterialTheme.colorScheme.background
                ) {
                    UserScreen(UserViewModel())
                }
            }
        }
    }
}

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun UserScreen(viewModel: UserViewModel) {
    val state by viewModel.state.collectAsState()

    Column(
        modifier = Modifier
            .fillMaxWidth()
            .padding(10.dp),
        horizontalAlignment = Alignment.CenterHorizontally
    ) {

        OutlinedTextField(
            label = { Text("Name") },
            value = state.name,
            onValueChange = {
                viewModel.updateProperty(state.copy(name = it))
            },
            isError = state.error?.property == "name"
        )

        Spacer(modifier = Modifier.height(10.dp))

        OutlinedTextField(
            label = { Text("Email") },
            value = state.email,
            onValueChange = {
                viewModel.updateProperty(state.copy(email = it))
            },
            isError = state.error?.property == "email"
        )

        Spacer(modifier = Modifier.height(10.dp))

        OutlinedTextField(
            label = { Text("Password") },
            value = state.password,
            onValueChange = {
                viewModel.updateProperty(state.copy(password = it))
            },
            isError = state.error?.property == "password"
        )

        Text(
            text = state.error?.message?:"",
            color =  MaterialTheme.colorScheme.error
        )

        Button(
            onClick = {
                viewModel.save()
            },
            content ={ Text("Save")}
        )

    }
}

class UserViewModel : ViewModel() {
    private val _state: MutableStateFlow<UserState> = MutableStateFlow(UserState())
    val state: StateFlow<UserState> = _state

    fun updateProperty(newState: UserState) {
        _state.update { newState }
    }

    fun save() {
        val stateValidator = ValidateState(UserState::class);
        val error = stateValidator.validate(state.value)

        _state.update {
            it.copy(error = error)
        }
    }
}

data class UserState(

    @property:NotEmptyValidation()
    val name: String = "",

    @property:EmailValidation()
    val email: String = "",

    @property:PasswordValidation()
    val password: String = "",

    val error: ValidationError? = null // <- we add this

)

@Composable
@Preview
fun UserScreenPreview() {
    AppTheme {
        Surface {
            UserScreen(viewModel = UserViewModel())
        }
    }
}

@Target( AnnotationTarget.PROPERTY)
annotation class NotEmptyValidation()

@Target( AnnotationTarget.PROPERTY)
annotation class EmailValidation()

@Target( AnnotationTarget.PROPERTY)
annotation class PasswordValidation()

data class ValidationError(
    val property: String = "",
    val message: String = ""
)

class ValidateState<State: Any>(
    // we are going to use reflection so we pass the KClass instance
    val kClass: KClass<State>
) {
    fun validate(state: State): ValidationError? {
        kClass.memberProperties.forEach {
            if (it.annotations.isEmpty())
                return@forEach

            val annotation = it.annotations[0];
            val property = it.name
            val value = it.get(state)

            if (annotation is NotEmptyValidation) {
                if (isEmpty(value)) {
                    return ValidationError(property, "Must fill this field")
                }
            }

            if (annotation is EmailValidation) {
                if (isNotEmail(value)) {
                    return ValidationError(property, "Not valid email")
                }

            }

            if (annotation is PasswordValidation) {
                 if (isNotPassword(value)) {
                    return ValidationError(property, "Not valid password")
                }
            }

        }

        return null
    }

    private fun isEmpty(value: Any?): Boolean {
        return value.toString().isEmpty()
    }

    private fun isNotEmail(value: Any?): Boolean{
       return !Regex("^[A-Za-z](.*)([@]{1})(.{1,})(\\.)(.{1,})")
           .matches(value.toString())
    }

    private fun isNotPassword(value: Any?): Boolean{
        return !Regex("^(?=.*[a-zA-Z])(?=.*\\d)[a-zA-Z\\d]{5,}$")
            .matches(value.toString());
    }

}
```

- Github: [https://github.com/FractalCodeRicardo](https://github.com/FractalCodeRicardo)

- Medium: [https://medium.com/@nosilverbullet](https://medium.com/@nosilverbullet )

- Web page: [https://programmingheadache.com](https://programmingheadache.com )

- Youtube: [https://www.youtube.com/@ProgrammingHeadache](https://www.youtube.com/@ProgrammingHeadache)

- Source Code: [https://github.com/FractalCodeRicardo/programmingheadache-misc/tree/main/custom-validation-annotation](https://github.com/FractalCodeRicardo/programmingheadache-misc/tree/main/custom-validation-annotation)
