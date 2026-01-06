# DataStore with Hilt Injection - Complete Tutorial

একটি beginner-friendly guide যেখানে আপনি শিখবেন কিভাবে DataStore এর সাথে Hilt Dependency Injection use করতে হয়।

## 📋 Table of Contents

- [কি শিখবেন এই Tutorial এ](#কি-শিখবেন-এই-tutorial-এ)
- [Prerequisites](#prerequisites)
- [Project Setup](#project-setup)
- [Method 1: Extension Property (Simple ⭐ Recommended)](#method-1-extension-property-simple--recommended)
- [Method 2: PreferenceDataStoreFactory (Alternative)](#method-2-preferencedatastorefactory-alternative)
- [Multiple DataStore Setup](#multiple-datastore-setup)
- [Usage Examples](#usage-examples)
- [Which Method to Choose](#which-method-to-choose)

---

## কি শিখবেন এই Tutorial এ

এই tutorial এ দুটি approach শেখানো হবে:

1. **Extension Property** - সহজ এবং কম code (বেশিরভাগ প্রজেক্টের জন্য perfect)
2. **PreferenceDataStoreFactory** - Alternative approach (উভয়ই একই কাজ করে)

আমরা **Multiple DataStore** setup করবো:
- **UserSettings DataStore** - Theme, Language, Font Size
- **Auth DataStore** - Token, User Info
- **App Config DataStore** - Onboarding, App Version

---

## Prerequisites

- Android Studio (latest version)
- Kotlin basics জানা থাকতে হবে
- Coroutines সম্পর্কে basic ধারণা

---

## Project Setup

### Step 1: Dependencies Add করুন

**build.gradle.kts (Project level):**
```kotlin
plugins {
    id("com.android.application") version "8.2.0" apply false
    id("org.jetbrains.kotlin.android") version "1.9.20" apply false
    id("com.google.dagger.hilt.android") version "2.51.1" apply false
}
```

**build.gradle.kts (Module level):**
```kotlin
plugins {
    id("com.android.application")
    id("org.jetbrains.kotlin.android")
    id("kotlin-kapt")
    id("com.google.dagger.hilt.android")
}

android {
    namespace = "com.example.datastoreapp"
    compileSdk = 34

    defaultConfig {
        applicationId = "com.example.datastoreapp"
        minSdk = 24
        targetSdk = 34
        versionCode = 1
        versionName = "1.0"
    }

    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_17
        targetCompatibility = JavaVersion.VERSION_17
    }
    
    kotlinOptions {
        jvmTarget = "17"
    }
}

dependencies {
    // AndroidX
    implementation("androidx.core:core-ktx:1.12.0")
    implementation("androidx.appcompat:appcompat:1.6.1")
    
    // DataStore Preferences - Key-Value data store করার জন্য (এটাই লাগবে)
    implementation("androidx.datastore:datastore-preferences:1.1.1")
    
    // Hilt - Dependency Injection এর জন্য
    implementation("com.google.dagger:hilt-android:2.51.1")
    kapt("com.google.dagger:hilt-compiler:2.51.1")
    
    // Coroutines - Asynchronous programming এর জন্য
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-android:1.8.0")
    
    // Lifecycle - ViewModel এবং lifecycle-aware components এর জন্য
    implementation("androidx.lifecycle:lifecycle-viewmodel-ktx:2.7.0")
    implementation("androidx.lifecycle:lifecycle-runtime-ktx:2.7.0")
}
```

### DataStore Libraries ব্যাখ্যা:

#### `datastore-preferences` (এটা use করবো ✅)
```kotlin
implementation("androidx.datastore:datastore-preferences:1.1.1")
```
**কি কাজ করে:**
- Key-Value pairs store করার জন্য (যেমন SharedPreferences এর মতো)
- Simple data types: String, Int, Boolean, Float, Long
- সবচেয়ে common use case
- আমরা এই tutorial এ এটাই use করবো

**কখন use করবেন:**
- ✅ App settings (theme, language)
- ✅ User preferences
- ✅ Simple configurations
- ✅ Auth tokens

#### `datastore-core` (Advanced use case এর জন্য ⚠️)
```kotlin
implementation("androidx.datastore:datastore-core:1.1.1")
```
**কি কাজ করে:**
- Custom data types store করার জন্য
- Protocol Buffers (Proto DataStore) use করতে হয়
- Complex data structures এর জন্য
- Type-safe কিন্তু setup জটিল

**কখন use করবেন:**
- ⚠️ Complex objects store করতে হলে
- ⚠️ Type-safety absolutely প্রয়োজন হলে
- ⚠️ Proto DataStore implement করতে হলে
- ⚠️ Custom serialization লাগলে

**এই tutorial এ `datastore-core` লাগবে না কারণ:**
- আমরা simple key-value pairs store করবো
- `datastore-preferences` যথেষ্ট
- Setup সহজ রাখতে চাই

### Step 2: Project Structure
```
app/
├── di/
│   ├── DataStoreModule.kt
│   ├── DataStoreExtensions.kt
│   └── Qualifiers.kt
│
├── data/
│   ├── local/
│   │   ├── UserSettingsDataStore.kt
│   │   ├── AuthDataStore.kt
│   │   └── AppConfigDataStore.kt
│   │
│   └── model/
│       ├── UserSettings.kt
│       ├── AuthData.kt
│       └── AppConfig.kt
│
└── MyApplication.kt
```

---

## Method 1: Extension Property (Simple ⭐ Recommended)

এই method টা **সবচেয়ে সহজ** এবং বেশিরভাগ প্রজেক্টের জন্য যথেষ্ট।

### সব Files একসাথে:

#### File 1: Qualifiers.kt
```kotlin
// di/Qualifiers.kt
package com.example.datastoreapp.di

import javax.inject.Qualifier

@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class UserSettingsDataStore

@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class AuthDataStore

@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class AppConfigDataStore
```

**কেন Qualifiers লাগে?**  
যখন multiple DataStore থাকে, Hilt কে বলতে হয় কোনটা কোথায় inject করবে।

---

#### File 2: DataStoreExtensions.kt
```kotlin
// di/DataStoreExtensions.kt
package com.example.datastoreapp.di

import android.content.Context
import androidx.datastore.core.DataStore
import androidx.datastore.preferences.core.Preferences
import androidx.datastore.preferences.preferencesDataStore

// Extension properties - এগুলো automatically singleton এবং thread-safe
val Context.userSettingsDataStore: DataStore<Preferences> by preferencesDataStore(
    name = "user_settings"
)

val Context.authDataStore: DataStore<Preferences> by preferencesDataStore(
    name = "auth_data"
)

val Context.appConfigDataStore: DataStore<Preferences> by preferencesDataStore(
    name = "app_config"
)
```

---

#### File 3: DataStoreModule.kt
```kotlin
// di/DataStoreModule.kt
package com.example.datastoreapp.di

import android.content.Context
import androidx.datastore.core.DataStore
import androidx.datastore.preferences.core.Preferences
import dagger.Module
import dagger.Provides
import dagger.hilt.InstallIn
import dagger.hilt.android.qualifiers.ApplicationContext
import dagger.hilt.components.SingletonComponent
import javax.inject.Singleton

@Module
@InstallIn(SingletonComponent::class)
object DataStoreModule {
    
    @Provides
    @Singleton
    @UserSettingsDataStore
    fun provideUserSettingsDataStore(
        @ApplicationContext context: Context
    ): DataStore<Preferences> {
        return context.userSettingsDataStore
    }
    
    @Provides
    @Singleton
    @AuthDataStore
    fun provideAuthDataStore(
        @ApplicationContext context: Context
    ): DataStore<Preferences> {
        return context.authDataStore
    }
    
    @Provides
    @Singleton
    @AppConfigDataStore
    fun provideAppConfigDataStore(
        @ApplicationContext context: Context
    ): DataStore<Preferences> {
        return context.appConfigDataStore
    }
}
```

---

#### File 4: UserSettings.kt
```kotlin
// data/model/UserSettings.kt
package com.example.datastoreapp.data.model

data class UserSettings(
    val isDarkTheme: Boolean = false,
    val language: String = "en",
    val fontSize: Int = 14
)
```

---

#### File 5: AuthData.kt
```kotlin
// data/model/AuthData.kt
package com.example.datastoreapp.data.model

data class AuthData(
    val accessToken: String = "",
    val userId: String = "",
    val isLoggedIn: Boolean = false
)
```

---

#### File 6: AppConfig.kt
```kotlin
// data/model/AppConfig.kt
package com.example.datastoreapp.data.model

data class AppConfig(
    val isOnboardingCompleted: Boolean = false,
    val appVersion: String = ""
)
```

---

#### File 7: UserSettingsDataStore.kt
```kotlin
// data/local/UserSettingsDataStore.kt
package com.example.datastoreapp.data.local

import androidx.datastore.core.DataStore
import androidx.datastore.preferences.core.Preferences
import androidx.datastore.preferences.core.booleanPreferencesKey
import androidx.datastore.preferences.core.edit
import androidx.datastore.preferences.core.emptyPreferences
import androidx.datastore.preferences.core.intPreferencesKey
import androidx.datastore.preferences.core.stringPreferencesKey
import com.example.datastoreapp.data.model.UserSettings
import com.example.datastoreapp.di.UserSettingsDataStore
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.catch
import kotlinx.coroutines.flow.map
import java.io.IOException
import javax.inject.Inject
import javax.inject.Singleton

@Singleton
class UserSettingsDataStoreWrapper @Inject constructor(
    @UserSettingsDataStore private val dataStore: DataStore<Preferences>
) {
    
    private object Keys {
        val THEME = booleanPreferencesKey("is_dark_theme")
        val LANGUAGE = stringPreferencesKey("language")
        val FONT_SIZE = intPreferencesKey("font_size")
    }
    
    // Read all settings
    val userSettings: Flow<UserSettings> = dataStore.data
        .catch { exception ->
            if (exception is IOException) {
                emit(emptyPreferences())
            } else {
                throw exception
            }
        }
        .map { preferences ->
            UserSettings(
                isDarkTheme = preferences[Keys.THEME] ?: false,
                language = preferences[Keys.LANGUAGE] ?: "en",
                fontSize = preferences[Keys.FONT_SIZE] ?: 14
            )
        }
    
    // Update theme
    suspend fun updateTheme(isDark: Boolean) {
        dataStore.edit { preferences ->
            preferences[Keys.THEME] = isDark
        }
    }
    
    // Update language
    suspend fun updateLanguage(language: String) {
        dataStore.edit { preferences ->
            preferences[Keys.LANGUAGE] = language
        }
    }
    
    // Update font size
    suspend fun updateFontSize(size: Int) {
        dataStore.edit { preferences ->
            preferences[Keys.FONT_SIZE] = size
        }
    }
    
    // Clear all
    suspend fun clearAll() {
        dataStore.edit { it.clear() }
    }
}
```

---

#### File 8: AuthDataStore.kt
```kotlin
// data/local/AuthDataStore.kt
package com.example.datastoreapp.data.local

import androidx.datastore.core.DataStore
import androidx.datastore.preferences.core.Preferences
import androidx.datastore.preferences.core.booleanPreferencesKey
import androidx.datastore.preferences.core.edit
import androidx.datastore.preferences.core.emptyPreferences
import androidx.datastore.preferences.core.stringPreferencesKey
import com.example.datastoreapp.data.model.AuthData
import com.example.datastoreapp.di.AuthDataStore
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.catch
import kotlinx.coroutines.flow.map
import java.io.IOException
import javax.inject.Inject
import javax.inject.Singleton

@Singleton
class AuthDataStoreWrapper @Inject constructor(
    @AuthDataStore private val dataStore: DataStore<Preferences>
) {
    
    private object Keys {
        val ACCESS_TOKEN = stringPreferencesKey("access_token")
        val USER_ID = stringPreferencesKey("user_id")
        val IS_LOGGED_IN = booleanPreferencesKey("is_logged_in")
    }
    
    // Read auth data
    val authData: Flow<AuthData> = dataStore.data
        .catch { exception ->
            if (exception is IOException) {
                emit(emptyPreferences())
            } else {
                throw exception
            }
        }
        .map { preferences ->
            AuthData(
                accessToken = preferences[Keys.ACCESS_TOKEN] ?: "",
                userId = preferences[Keys.USER_ID] ?: "",
                isLoggedIn = preferences[Keys.IS_LOGGED_IN] ?: false
            )
        }
    
    // Save auth tokens
    suspend fun saveAuthData(token: String, userId: String) {
        dataStore.edit { preferences ->
            preferences[Keys.ACCESS_TOKEN] = token
            preferences[Keys.USER_ID] = userId
            preferences[Keys.IS_LOGGED_IN] = true
        }
    }
    
    // Clear auth data (logout)
    suspend fun clearAuthData() {
        dataStore.edit { it.clear() }
    }
}
```

---

#### File 9: AppConfigDataStore.kt
```kotlin
// data/local/AppConfigDataStore.kt
package com.example.datastoreapp.data.local

import androidx.datastore.core.DataStore
import androidx.datastore.preferences.core.Preferences
import androidx.datastore.preferences.core.booleanPreferencesKey
import androidx.datastore.preferences.core.edit
import androidx.datastore.preferences.core.emptyPreferences
import androidx.datastore.preferences.core.stringPreferencesKey
import com.example.datastoreapp.data.model.AppConfig
import com.example.datastoreapp.di.AppConfigDataStore
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.catch
import kotlinx.coroutines.flow.map
import java.io.IOException
import javax.inject.Inject
import javax.inject.Singleton

@Singleton
class AppConfigDataStoreWrapper @Inject constructor(
    @AppConfigDataStore private val dataStore: DataStore<Preferences>
) {
    
    private object Keys {
        val ONBOARDING_COMPLETED = booleanPreferencesKey("onboarding_completed")
        val APP_VERSION = stringPreferencesKey("app_version")
    }
    
    // Read app config
    val appConfig: Flow<AppConfig> = dataStore.data
        .catch { exception ->
            if (exception is IOException) {
                emit(emptyPreferences())
            } else {
                throw exception
            }
        }
        .map { preferences ->
            AppConfig(
                isOnboardingCompleted = preferences[Keys.ONBOARDING_COMPLETED] ?: false,
                appVersion = preferences[Keys.APP_VERSION] ?: ""
            )
        }
    
    // Mark onboarding as completed
    suspend fun setOnboardingCompleted() {
        dataStore.edit { preferences ->
            preferences[Keys.ONBOARDING_COMPLETED] = true
        }
    }
    
    // Update app version
    suspend fun updateAppVersion(version: String) {
        dataStore.edit { preferences ->
            preferences[Keys.APP_VERSION] = version
        }
    }
}
```

---

#### File 10: MyApplication.kt
```kotlin
// MyApplication.kt
package com.example.datastoreapp

import android.app.Application
import dagger.hilt.android.HiltAndroidApp

@HiltAndroidApp
class MyApplication : Application()
```

**AndroidManifest.xml এ add করুন:**
```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <application
        android:name=".MyApplication"
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/Theme.AppCompat">
        
        <!-- Your activities here -->
        
    </application>

</manifest>
```

### ✅ Done! Method 1 Complete

---

## Method 2: PreferenceDataStoreFactory (Alternative)

এই method টাও একই কাজ করে, শুধু DataStore create করার way আলাদা।

### পার্থক্য কি?
```kotlin
// Method 1: Extension Property (Recommended)
val Context.dataStore by preferencesDataStore(name = "settings")

// Method 2: Factory (Alternative - একই কাজ)
PreferenceDataStoreFactory.create {
    context.preferencesDataStoreFile("settings")
}
```

### কোন files different?

**Files 1-2, 4-10 একই থাকবে Method 1 এর মতো।**  
**শুধু File 3 (DataStoreModule.kt) different হবে:**

#### File 3 (Modified): DataStoreModule.kt
```kotlin
// di/DataStoreModule.kt (Method 2)
package com.example.datastoreapp.di

import android.content.Context
import androidx.datastore.core.DataStore
import androidx.datastore.preferences.core.PreferenceDataStoreFactory
import androidx.datastore.preferences.core.Preferences
import androidx.datastore.preferences.preferencesDataStoreFile
import dagger.Module
import dagger.Provides
import dagger.hilt.InstallIn
import dagger.hilt.android.qualifiers.ApplicationContext
import dagger.hilt.components.SingletonComponent
import javax.inject.Singleton

@Module
@InstallIn(SingletonComponent::class)
object DataStoreModule {
    
    @Provides
    @Singleton
    @UserSettingsDataStore
    fun provideUserSettingsDataStore(
        @ApplicationContext context: Context
    ): DataStore<Preferences> {
        return PreferenceDataStoreFactory.create(
            produceFile = {
                context.preferencesDataStoreFile("user_settings")
            }
        )
    }
    
    @Provides
    @Singleton
    @AuthDataStore
    fun provideAuthDataStore(
        @ApplicationContext context: Context
    ): DataStore<Preferences> {
        return PreferenceDataStoreFactory.create(
            produceFile = {
                context.preferencesDataStoreFile("auth_data")
            }
        )
    }
    
    @Provides
    @Singleton
    @AppConfigDataStore
    fun provideAppConfigDataStore(
        @ApplicationContext context: Context
    ): DataStore<Preferences> {
        return PreferenceDataStoreFactory.create(
            produceFile = {
                context.preferencesDataStoreFile("app_config")
            }
        )
    }
}
```

**Note:** Method 2 এ File 2 (DataStoreExtensions.kt) লাগবে না কারণ আমরা extension properties use করছি না।

### ✅ Done! Method 2 Complete

---

## Multiple DataStore Setup

### কেন Multiple DataStore ব্যবহার করবো?

**❌ Bad Practice - একটা DataStore এ সব:**
```kotlin
SingleDataStore {
    // User preferences
    theme, language, fontSize,
    
    // Auth data (sensitive!)
    token, userId, password,
    
    // App config
    onboarding, version, lastSync
}
```

**Problems:**
- সব mixed up
- Security risk (sensitive data আলাদা না)
- Performance issue (বড় file slow)
- Maintenance কঠিন

**✅ Good Practice - আলাদা আলাদা:**
```kotlin
UserSettingsDataStore {
    theme, language, fontSize
}

AuthDataStore {
    token, userId
}

AppConfigDataStore {
    onboarding, version
}
```

**Benefits:**
- ✅ Better organization
- ✅ Security (sensitive data আলাদা)
- ✅ Performance (ছোট files fast)
- ✅ Easy maintenance

### Qualifiers কিভাবে কাজ করে?

**Without Qualifiers (Hilt confused!):**
```kotlin
class MyClass @Inject constructor(
    private val dataStore1: DataStore<Preferences>, // ??? Which one?
    private val dataStore2: DataStore<Preferences>  // ??? Which one?
)
```

**With Qualifiers (Clear!):**
```kotlin
class MyClass @Inject constructor(
    @UserSettingsDataStore private val settingsStore: DataStore<Preferences>, // ✅ Clear
    @AuthDataStore private val authStore: DataStore<Preferences>              // ✅ Clear
)
```

---

## Usage Examples

### Example 1: ViewModel এ Use করা
```kotlin
// presentation/SettingsViewModel.kt
package com.example.datastoreapp.presentation

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.example.datastoreapp.data.local.UserSettingsDataStoreWrapper
import com.example.datastoreapp.data.model.UserSettings
import dagger.hilt.android.lifecycle.HiltViewModel
import kotlinx.coroutines.flow.SharingStarted
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.stateIn
import kotlinx.coroutines.launch
import javax.inject.Inject

@HiltViewModel
class SettingsViewModel @Inject constructor(
    private val userSettingsDataStore: UserSettingsDataStoreWrapper
) : ViewModel() {
    
    // Settings কে StateFlow হিসেবে collect করা
    val userSettings: StateFlow<UserSettings> = 
        userSettingsDataStore.userSettings.stateIn(
            scope = viewModelScope,
            started = SharingStarted.WhileSubscribed(5000),
            initialValue = UserSettings()
        )
    
    // Theme update করা
    fun updateTheme(isDark: Boolean) {
        viewModelScope.launch {
            userSettingsDataStore.updateTheme(isDark)
        }
    }
    
    // Language update করা
    fun updateLanguage(language: String) {
        viewModelScope.launch {
            userSettingsDataStore.updateLanguage(language)
        }
    }
    
    // Font size update করা
    fun updateFontSize(size: Int) {
        viewModelScope.launch {
            userSettingsDataStore.updateFontSize(size)
        }
    }
}
```

---

### Example 2: Multiple DataStore একসাথে Use করা
```kotlin
// data/repository/UserRepository.kt
package com.example.datastoreapp.data.repository

import com.example.datastoreapp.data.local.AuthDataStoreWrapper
import com.example.datastoreapp.data.local.UserSettingsDataStoreWrapper
import kotlinx.coroutines.flow.first
import javax.inject.Inject
import javax.inject.Singleton

@Singleton
class UserRepository @Inject constructor(
    private val authDataStore: AuthDataStoreWrapper,
    private val userSettingsDataStore: UserSettingsDataStoreWrapper
) {
    
    // Login করা
    suspend fun login(token: String, userId: String): Result<Unit> {
        return try {
            authDataStore.saveAuthData(token, userId)
            Result.success(Unit)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
    
    // Logout করা - দুটো DataStore clear করা
    suspend fun logout(): Result<Unit> {
        return try {
            authDataStore.clearAuthData()
            userSettingsDataStore.clearAll()
            Result.success(Unit)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
    
    // Check if logged in
    suspend fun isLoggedIn(): Boolean {
        return authDataStore.authData.first().isLoggedIn
    }
    
    // Get user's preferred language
    suspend fun getUserLanguage(): String {
        return userSettingsDataStore.userSettings.first().language
    }
}
```

---

### Example 3: Activity তে Use করা
```kotlin
// presentation/MainActivity.kt
package com.example.datastoreapp.presentation

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.viewModels
import androidx.lifecycle.Lifecycle
import androidx.lifecycle.lifecycleScope
import androidx.lifecycle.repeatOnLifecycle
import dagger.hilt.android.AndroidEntryPoint
import kotlinx.coroutines.launch

@AndroidEntryPoint
class MainActivity : ComponentActivity() {
    
    private val viewModel: SettingsViewModel by viewModels()
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // Settings collect করা
        lifecycleScope.launch {
            repeatOnLifecycle(Lifecycle.State.STARTED) {
                viewModel.userSettings.collect { settings ->
                    // UI update করা based on settings
                    applyTheme(settings.isDarkTheme)
                    applyLanguage(settings.language)
                    applyFontSize(settings.fontSize)
                }
            }
        }
    }
    
    private fun applyTheme(isDark: Boolean) {
        // Your theme change logic
    }
    
    private fun applyLanguage(language: String) {
        // Your language change logic
    }
    
    private fun applyFontSize(size: Int) {
        // Your font size change logic
    }
    
    // Button click করলে theme toggle করা
    private fun onThemeButtonClick() {
        val currentTheme = viewModel.userSettings.value.isDarkTheme
        viewModel.updateTheme(!currentTheme)
    }
}
```

---

### Example 4: UseCase এ Multiple DataStore Combine করা
```kotlin
// domain/usecase/GetAppStateUseCase.kt
package com.example.datastoreapp.domain.usecase

import com.example.datastoreapp.data.local.AppConfigDataStoreWrapper
import com.example.datastoreapp.data.local.AuthDataStoreWrapper
import com.example.datastoreapp.data.local.UserSettingsDataStoreWrapper
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.combine
import javax.inject.Inject

data class AppState(
    val isLoggedIn: Boolean,
    val isDarkTheme: Boolean,
    val isOnboardingCompleted: Boolean,
    val language: String
)

class GetAppStateUseCase @Inject constructor(
    private val userSettingsDataStore: UserSettingsDataStoreWrapper,
    private val authDataStore: AuthDataStoreWrapper,
    private val appConfigDataStore: AppConfigDataStoreWrapper
) {
    
    // তিনটা DataStore থেকে data combine করে একটা Flow return করা
    operator fun invoke(): Flow<AppState> {
        return combine(
            authDataStore.authData,
            userSettingsDataStore.userSettings,
            appConfigDataStore.appConfig
        ) { auth, settings, config ->
            AppState(
                isLoggedIn = auth.isLoggedIn,
                isDarkTheme = settings.isDarkTheme,
                isOnboardingCompleted = config.isOnboardingCompleted,
                language = settings.language
            )
        }
    }
}
```

---

## Which Method to Choose

### Side by Side Comparison
```kotlin
// ============================================
// Method 1: Extension Property (Recommended)
// ============================================

// 1. Extension তৈরি করা
val Context.dataStore by preferencesDataStore(name = "settings")

// 2. Module এ provide করা
@Provides
fun provideDataStore(@ApplicationContext context: Context) = 
    context.dataStore

// Total Lines: 2-3 lines per DataStore
```
```kotlin
// ============================================
// Method 2: Factory (Alternative)
// ============================================

// 1. Directly Factory use করা Module এ
@Provides
fun provideDataStore(@ApplicationContext context: Context) = 
    PreferenceDataStoreFactory.create {
        context.preferencesDataStoreFile("settings")
    }

// Total Lines: 3-4 lines per DataStore
```

### কোনটা কখন?

| Aspect | Method 1 (Extension) | Method 2 (Factory) |
|--------|---------------------|-------------------|
| **Code Length** | ✅ Shorter | ⚠️ Slightly Longer |
| **Readability** | ✅ Very Clean | ⚠️ Good |
| **Industry Standard** | ✅ Yes | ✅ Yes |
| **Performance** | ✅ Same | ✅ Same |
| **Flexibility** | ⚠️ Standard | ✅ More Options |
| **Recommended** | ✅ YES | ⚠️ Alternative |

### 🌟 Final Recommendation

**বেশিরভাগ সময় Method 1 (Extension Property) use করুন কারণ:**
- ✅ Clean এবং concise code
- ✅ Kotlin idioms follow করে
- ✅ Less boilerplate
- ✅ Industry এ বেশি প্রচলিত

**Method 2 (Factory) শুধু তখনই যদি:**
- আপনার specific requirements আছে
- অথবা আপনি Factory pattern prefer করেন

---

## Complete Architecture Flow
```
┌─────────────────────────────────────────────────────────────┐
│                    Hilt Module (DI)                         │
│              @InstallIn(SingletonComponent)                 │
└──────┬────────────────────┬────────────────────┬────────────┘
       │                    │                    │
       │ @UserSettings      │ @Auth              │ @AppConfig
       │ DataStore          │ DataStore          │ DataStore
       ▼                    ▼                    ▼
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│ DataStore    │    │ DataStore    │    │ DataStore    │
│ (Settings)   │    │ (Auth)       │    │ (Config)     │
│              │    │              │    │              │
│ File:        │    │ File:        │    │ File:        │
│ user_        │    │ auth_        │    │ app_         │
│ settings     │    │ data         │    │ config       │
└──────┬───────┘    └──────┬───────┘    └──────┬───────┘
       │                    │                    │
       │ Injected into      │                    │
       ▼                    ▼                    ▼
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│ Wrapper      │    │ Wrapper      │    │ Wrapper      │
│ Class        │    │ Class        │    │ Class        │
│              │    │              │    │              │
│ • Get Data   │    │ • Get Data   │    │ • Get Data   │
│ • Update     │    │ • Update     │    │ • Update     │
│ • Clear      │    │ • Clear      │    │ • Clear      │
└──────┬───────┘    └──────┬───────┘    └──────┬───────┘
       │                    │                    │
       └────────────────────┴────────────────────┘
                            │
                            │ Used in
                            ▼
                 ┌──────────────────────┐
                 │  ViewModel/Repository│
                 │                      │
                 │  • Business Logic    │
                 │  • Data Validation   │
                 │  • Combine Data      │
                 └──────────┬───────────┘
                            │
                            │ Collected by
                            ▼
                 ┌──────────────────────┐
                 │    UI Layer          │
                 │   (Activity/Fragment)│
                 │                      │
                 │  • Display Data      │
                 │  • User Interaction  │
                 └──────────────────────┘
```

---

## Key Takeaways

### 1. Dependencies
```kotlin
// শুধু এটা লাগবে simple key-value storage এর জন্য
implementation("androidx.datastore:datastore-preferences:1.1.1")

// এটা লাগবে না (complex types এর জন্য)
// implementation("androidx.datastore:datastore-core:1.1.1")
```

### 2. Multiple DataStore
```kotlin
// আলাদা আলাদা DataStore = Better organization
UserSettingsDataStore  // UI preferences
AuthDataStore          // Login data
AppConfigDataStore     // App state
```

### 3. Qualifiers Must
```kotlin
// Multiple DataStore থাকলে Qualifier অবশ্যই লাগবে
@UserSettingsDataStore
@AuthDataStore
@AppConfigDataStore
```

### 4. Method Choice
```kotlin
// Method 1: Extension Property (Recommended)
val Context.dataStore by preferencesDataStore(name = "settings")

// Method 2: Factory (Alternative - same result)
PreferenceDataStoreFactory.create { ... }
```

### 5. Architecture Pattern
```kotlin
// এই flow follow করুন
Module → DataStore → Wrapper → Repository/ViewModel → UI
```

---

## Common Mistakes to Avoid

### ❌ Mistake 1: No Qualifiers with Multiple DataStores
```kotlin
// Wrong - Hilt confused!
@Provides
fun provideDataStore1(...): DataStore<Preferences> { }

@Provides
fun provideDataStore2(...): DataStore<Preferences> { }
```
```kotlin
// Correct - Use Qualifiers
@Provides
@UserSettingsDataStore
fun provideDataStore1(...): DataStore<Preferences> { }

@Provides
@AuthDataStore
fun provideDataStore2(...): DataStore<Preferences> { }
```

### ❌ Mistake 2: Same File Name for Multiple DataStores
```kotlin
// Wrong - File conflict!
val Context.dataStore1 by preferencesDataStore(name = "app_data")
val Context.dataStore2 by preferencesDataStore(name = "app_data")
```
```kotlin
// Correct - Unique names
val Context.settingsStore by preferencesDataStore(name = "user_settings")
val Context.authStore by preferencesDataStore(name = "auth_data")
```

### ❌ Mistake 3: Forgetting Application Class
```kotlin
// Wrong - App will crash!
class MyApplication : Application() {
    // No @HiltAndroidApp annotation
}
```
```kotlin
// Correct
@HiltAndroidApp
class MyApplication : Application()
```

### ❌ Mistake 4: Direct DataStore Access without Wrapper
```kotlin
// Wrong - Repeated code everywhere
class MyViewModel @Inject constructor(
    @UserSettingsDataStore private val dataStore: DataStore<Preferences>
) {
    // Writing same read/write logic everywhere
}
```
```kotlin
// Correct - Use Wrapper
class MyViewModel @Inject constructor(
    private val userSettingsDataStore: UserSettingsDataStoreWrapper
) {
    // Clean API
}
```

---

## Summary

### যা শিখলেন:

✅ DataStore setup with Hilt  
✅ Multiple DataStore configuration  
✅ Qualifier ব্যবহার করে DataStore distinguish করা  
✅ Wrapper class pattern  
✅ দুটি different approach (Extension vs Factory)  
✅ Real-world usage examples  
✅ Complete architecture flow  

### Dependencies Summary:
```kotlin
// Only this is needed for this tutorial
implementation("androidx.datastore:datastore-preferences:1.1.1")

// datastore-core শুধু advanced use cases এর জন্য
// (Proto DataStore, custom types)
// This tutorial doesn't need it
```

### Method Recommendation:

🌟 **Use Method 1 (Extension Property)**
- Clean code
- Industry standard
- Easy to understand

⚙️ **Method 2 (Factory) is optional**
- Same result
- Alternative syntax
- Slightly more verbose

### Next Steps:

1. আপনার project এ implement করুন
2. Method 1 দিয়ে start করুন
3. প্রয়োজন মতো multiple DataStore add করুন
4. Wrapper classes use করে clean API maintain করুন

---

## Resources

- [Official DataStore Documentation](https://developer.android.com/topic/libraries/architecture/datastore)
- [Hilt Documentation](https://dagger.dev/hilt/)
- [Kotlin Coroutines Guide](https://kotlinlang.org/docs/coroutines-guide.html)

---

**Happy Coding! 🚀**

---

**Created by:** Android Developers Community  
**Last Updated:** 2025  
**License:** MIT
