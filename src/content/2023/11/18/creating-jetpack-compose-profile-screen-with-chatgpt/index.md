---
title: "Creating Jetpack Compose Profile Screen with ChatGPT"
date: 2023-11-18
categories: 
  - "chatgpt"
  - "jetpack-compose"
  - "kotlin"
image: "/images/image-6.png"
---

I want a profile screen with three fields:

- Profile picture

- Name

- Email

I want the full functionality, so I am going to ask for these parts of the architecture:

- The view with the composables

- The view model

- The data access using room

## View

I started with a simple pompt like this

![](/images/image-6.png)

This is the code that it gave me

```kotlin
package com.example.profile_screen.screens

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.foundation.Image
import androidx.compose.foundation.background
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.shape.CircleShape
import androidx.compose.material3.*
import androidx.compose.material3.icons.Icons
import androidx.compose.material3.icons.filled.MailOutline
import androidx.compose.material3.icons.filled.Person
import androidx.compose.material3.icons.filled.Save
import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier
import androidx.compose.ui.draw.clip
import androidx.compose.ui.layout.ContentScale
import androidx.compose.ui.res.painterResource
import androidx.compose.ui.tooling.preview.Preview
import androidx.compose.ui.unit.dp

data class UserProfile(val name: String, val email: String, val profileImageRes: Int)

@Composable
fun ProfileScreen(userProfile: UserProfile) {
    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp)
    ) {
        // Profile Image
        Image(
            painter = painterResource(id = userProfile.profileImageRes),
            contentDescription = null,
            modifier = Modifier
                .size(120.dp)
                .clip(CircleShape)
                .background(MaterialTheme.colorScheme.primary)
        )

        Spacer(modifier = Modifier.height(16.dp))

        // Name Field
        OutlinedTextField(
            value = userProfile.name,
            onValueChange = { /*TODO*/ },
            label = { Text("Name") },
            leadingIcon = { Icon(imageVector = Icons.Default.Person, contentDescription = null) },
            modifier = Modifier
                .fillMaxWidth()
                .padding(8.dp)
        )

        Spacer(modifier = Modifier.height(16.dp))

        // Email Field
        OutlinedTextField(
            value = userProfile.email,
            onValueChange = { /*TODO*/ },
            label = { Text("Email") },
            leadingIcon = { Icon(imageVector = Icons.Default.MailOutline, contentDescription = null) },
            modifier = Modifier
                .fillMaxWidth()
                .padding(8.dp)
        )

        Spacer(modifier = Modifier.height(16.dp))

        // Save Button
        Button(
            onClick = { /*TODO*/ },
            modifier = Modifier
                .fillMaxWidth()
                .height(50.dp)
        ) {
            Icon(imageVector = Icons.Default.Save, contentDescription = null)
            Spacer(modifier = Modifier.width(8.dp))
            Text("Save")
        }
    }
}

@Preview(showBackground = true)
@Composable
fun ProfileScreenPreview() {
    val userProfile = UserProfile(
        name = "John Doe",
        email = "john.doe@example.com",
        profileImageRes = R.drawable.ic_launcher_foreground // Replace with your actual image resource
    )
    ProfileScreen(userProfile = userProfile)
}
```

I had to fix the imports of the icons, one of the icon didn't exists. Fixing that this is how to looks like:

![](/images/image-7.png)

  
Now I want the the user select the profile picture.

![](/images/image-8.png)

It changed the profile image field and add a functión to select an image. This is the new function:

```kotlin
var userProfile by remember { mutableStateOf(userProfile) }
val getContent = rememberLauncherForActivityResult(
    contract = ActivityResultContracts.GetContent()) { uri: Uri? ->
    uri?.let {
        userProfile = userProfile.copy(profileImageUri = it)
    }
}
```

And the profile image field now calls the _getContent_ function

```kotlin
    Box(
            modifier = Modifier
                .size(120.dp)
                .clip(CircleShape)
                .background(MaterialTheme.colorScheme.primary)
                .clickable {
                    getContent.launch("image/*") // Calls getContent
                },
            contentAlignment = Alignment.Center
        ) {
            if (userProfile.profileImageUri != null) {
                Image(
                    painter = painterResource(id = R.drawable.ic_launcher_foreground),
                    contentDescription = null,
                    contentScale = ContentScale.Crop,
                    modifier = Modifier
                        .fillMaxSize()
                        .clip(CircleShape)
                )
            } else {
                Icon(
                    imageVector = Icons.Default.Upload,
                    contentDescription = null,
                    modifier = Modifier.size(48.dp),
                    tint = MaterialTheme.colorScheme.onPrimary
                )
            }
        }
```

The code did not show the selected image, so I added a new dependency and changed some code  

```kotlin
implementation("io.coil-kt:coil-compose:2.5.0")
```

```kotlin
if (userProfile.profileImageUri != null) {
    Image(
        // I replace this line
        //painter = painterResource(id = R.drawable.ic_launcher_foreground),
        painter = rememberAsyncImagePainter(
            model = userProfile.profileImageUri   // or ht
        ),
        contentDescription = null,
        contentScale = ContentScale.Crop,
        modifier = Modifier
            .fillMaxSize()
            .clip(CircleShape)
    )
}
```

![](/images/ProfileScreenPickImage.gif)

It was necesary to add permissions to read the image, so I added a line when the image is picked

```kotlin
uri?.let {
  //Permissions to read the uri
  context.contentResolver.takePersistableUriPermission(uri, Intent.FLAG_GRANT_READ_URI_PERMISSION )

  //userProfile = userProfile.copy(profileImageUri = it)
  viewModel.setProfileImageUri(it.toString())
}
```

## View Model

For the ViewModel, this was my prompt

![](/images/image-9.png)

And ChatGPT gave me this code

```kotlin
import android.net.Uri
import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.launch

data class UserProfileState(
    val name: String = "",
    val email: String = "",
    val profileImageUri: Uri? = null
)

class ProfileViewModel : ViewModel() {

    private val _userProfileState = MutableStateFlow(UserProfileState())
    val userProfileState: StateFlow<UserProfileState> get() = _userProfileState

    fun setName(name: String) {
        _userProfileState.value = _userProfileState.value.copy(name = name)
    }

    fun setEmail(email: String) {
        _userProfileState.value = _userProfileState.value.copy(email = email)
    }

    fun setProfileImageUri(uri: Uri?) {
        _userProfileState.value = _userProfileState.value.copy(profileImageUri = uri)
    }

    fun saveUserProfile() {
        // Implement your save logic here
        viewModelScope.launch {
            // For example, you can save the user profile to a repository or database
            // userRepository.saveUserProfile(_userProfileState.value)
        }
    }
}
```

Nowadays the prefered way to update the view model is using the update method, so I changed it. I made some renames too, and I change the Uri parameter for a string because it is easier to store in the database.

```kotlin

class ProfileViewModel : ViewModel() {

    private val _state = MutableStateFlow(UserProfileState())
    val state: StateFlow<UserProfileState> = _state

    fun setName(name: String) {
        _state.update { it.copy(name = name) }
    }

    fun setEmail(email: String) {
        _state.update { it.copy(email = email) }
    }

    fun setProfileImageUri(uri: Uri?) {
        _state.update { it.copy(image = uri.toString()) }
    }

    fun saveUserProfile() {
        // Implement your save logic here
        viewModelScope.launch {
            // For example, you can save the user profile to a repository or database
            // userRepository.saveUserProfile(_userProfileState.value)
        }
    }
}
```

## Data access

First I'm going to create the data model and and data access objects

![](/images/image-10.png)

ChatGPT did a great job! I didn't have to ask for the Dao neither for the database class. It gave me this code

```kotlin

@Entity(tableName = "user_profiles")
data class UserProfileEntity(
    @PrimaryKey(autoGenerate = true)
    val id: Long = 0,
    val name: String,
    val image: String? = null,
    val email: String
)

@Dao
interface UserProfileDao {

    @Insert
    suspend fun insertUserProfile(userProfile: UserProfileEntity)

    @Query("SELECT * FROM user_profiles WHERE id = :userId")
    suspend fun getUserProfile(userId: Long): UserProfileEntity?

}

@Database(entities = [UserProfileEntity::class], version = 1)
abstract class AppDatabase : RoomDatabase() {
    abstract fun userProfileDao(): UserProfileDao
}
```

I'm going to make changes acoording my needs

```kotlin
@Entity(tableName = "profile")
data class Profile(
    @PrimaryKey(autoGenerate = true)
    val id: Long = 0,
    val name: String,
    val image: String? = null,
    val email: String
)

@Dao
interface ProfileDao {
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun save(profile: Profile)

    @Query("SELECT * FROM profile")
    suspend fun getAll(): List<Profile>
}

@Database(entities = [Profile::class], version = 1)
abstract class AppDatabase : RoomDatabase() {
    abstract fun profileDao(): ProfileDao
}
```

## Glue the parts

To add the database to the ViewModel just add two simple methods

```kotlin
    fun load() {
        viewModelScope.launch {
            val profiles = database.profileDao().getAll()

            if (profiles.isNotEmpty()) {
                val profile = profiles.first()

                _state.update {
                    it.copy(
                        name = profile.name,
                        image = null,
                        email = profile.email,
                        id = profile.id
                    )
                }

                setProfileImageUri(profile.image)
            }

        }
    }
    
    fun saveUserProfile() {
        viewModelScope.launch {
            val state = state.value
            var profile = Profile(
                id = state.id,
                name = state.name,
                email = state.email,
                image = state.image
            )

            database.profileDao().save(profile)
        }
    }
```

And I pass the database in the constructor

```kotlin
class ProfileViewModel(private val database: AppDatabase) : ViewModel()
```

Then I add the view model in the screen. For example to bind the email field with the view model I change the value and onValueChange properties of the composable

```kotlin
val state by viewModel.state.collectAsState()

OutlinedTextField(
    value = state.email, // Bind view Model
    onValueChange = {
        viewModel.setEmail(it)
    },
    label = { Text("Email") },
    leadingIcon = { Icon(
        imageVector = Icons.Default.MailOutline, 
        contentDescription = null) 
                  },
    modifier = Modifier
        .fillMaxWidth()
        .padding(8.dp)
)
```

GitHub Code: [https://github.com/FractalCodeRicardo/profile-screen](https://github.com/FractalCodeRicardo/profile-screen)
