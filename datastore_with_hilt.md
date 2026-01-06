# DataStore with Hilt Injection - Complete Tutorial

A comprehensive guide to implementing DataStore with Hilt dependency injection in Android, covering both Factory and Extension Property approaches with Wrapper Classes.

## 📋 Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Project Setup](#project-setup)
- [Method 1: Using PreferenceDataStoreFactory](#method-1-using-preferencedatastorefactory)
- [Method 2: Using Extension Property](#method-2-using-extension-property-without-factory)
- [Usage Examples](#usage-examples)
- [Testing](#testing)
- [Comparison](#comparison)
- [Best Practices](#best-practices)

---

## Overview

This tutorial demonstrates two approaches for injecting DataStore with Hilt:

1. **PreferenceDataStoreFactory** - Full control with custom configurations
2. **Extension Property (Delegation)** - Simpler approach with less boilerplate

Both methods use Wrapper Classes for better encapsulation and maintainability.

---

## Prerequisites

- Android Studio Arctic Fox or later
- Minimum SDK 24
- Kotlin 1.9+
- Basic knowledge of:
  - Kotlin Coroutines
  - Hilt Dependency Injection
  - DataStore Preferences

---

## Project Setup

### 1. Add Dependencies

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
    // AndroidX Core
    implementation("androidx.core:core-ktx:1.12.0")
    implementation("androidx.appcompat:appcompat:1.6.1")
    
    // DataStore
    implementation("androidx.datastore:datastore-preferences:1.1.1")
    implementation("androidx.datastore:datastore-core:1.1.1")
    
    // Hilt
    implementation("com.google.dagger:hilt-android:2.51.1")
    kapt("com.google.dagger:hilt-compiler:2.51.1")
    
    // Coroutines
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-android:1.8.0")
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.8.0")
    
    // Lifecycle
    implementation("androidx.lifecycle:lifecycle-runtime-ktx:2.7.0")
    implementation("androidx.lifecycle:lifecycle-viewmodel-ktx:2.7.0")
    
    // Testing
    testImplementation("junit:junit:4.13.2")
    testImplementation("org.jetbrains.kotlinx:kotlinx-coroutines-test:1.8.0")
    androidTestImplementation("androidx.test.ext:junit:1.1.5")
}
```

### 2. Project Structure
```
app/
├── src/
│   └── main/
│       ├── kotlin/
│       │   └── com/example/datastoreapp/
│       │       ├── di/
│       │       │   ├── DataStoreModule.kt
│       │       │   └── DataStoreQualifiers.kt
│       │       │
│       │       ├── data/
│       │       │   ├── local/
│       │       │   │   └── datastore/
│       │       │   │       ├── UserSettingsDataStore.kt
│       │       │   │       ├── AuthDataStore.kt
│       │       │   │       └── AppConfigDataStore.kt
│       │       │   │
│       │       │   └── model/
│       │       │       ├── UserSettings.kt
│       │       │       ├── AuthData.kt
│       │       │       └── AppConfig.kt
│       │       │
│       │       └── MyApplication.kt
│       │
│       └── AndroidManifest.xml
```

---

## Method 1: Using PreferenceDataStoreFactory

### Step 1: Create Qualifiers
```kotlin
// di/DataStoreQualifiers.kt
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

### Step 2: Create Hilt Module with Factory
```kotlin
// di/DataStoreModule.kt
package com.example.datastoreapp.di

import android.content.Context
import androidx.datastore.core.DataStore
import androidx.datastore.core.handlers.ReplaceFileCorruptionHandler
import androidx.datastore.preferences.core.PreferenceDataStoreFactory
import androidx.datastore.preferences.core.Preferences
import androidx.datastore.preferences.core.emptyPreferences
import androidx.datastore.preferences.preferencesDataStoreFile
import dagger.Module
import dagger.Provides
import dagger.hilt.InstallIn
import dagger.hilt.android.qualifiers.ApplicationContext
import dagger.hilt.components.SingletonComponent
import kotlinx.coroutines.CoroutineScope
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.SupervisorJob
import javax.inject.Singleton

@Module
@InstallIn(SingletonComponent::class)
object DataStoreModule {
    
    private const val USER_SETTINGS_NAME = "user_settings"
    private const val AUTH_DATA_NAME = "auth_data"
    private const val APP_CONFIG_NAME = "app_config"
    
    @Provides
    @Singleton
    @UserSettingsDataStore
    fun provideUserSettingsDataStore(
        @ApplicationContext context: Context
    ): DataStore<Preferences> {
        return PreferenceDataStoreFactory.create(
            corruptionHandler = ReplaceFileCorruptionHandler(
                produceNewData = { emptyPreferences() }
            ),
            scope = CoroutineScope(Dispatchers.IO + SupervisorJob()),
            produceFile = {
                context.preferencesDataStoreFile(USER_SETTINGS_NAME)
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
            corruptionHandler = ReplaceFileCorruptionHandler(
                produceNewData = { emptyPreferences() }
            ),
            scope = CoroutineScope(Dispatchers.IO + SupervisorJob()),
            produceFile = {
                context.preferencesDataStoreFile(AUTH_DATA_NAME)
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
            corruptionHandler = ReplaceFileCorruptionHandler(
                produceNewData = { emptyPreferences() }
            ),
            scope = CoroutineScope(Dispatchers.IO + SupervisorJob()),
            produceFile = {
                context.preferencesDataStoreFile(APP_CONFIG_NAME)
            }
        )
    }
}
```

### Step 3: Create Data Models
```kotlin
// data/model/UserSettings.kt
package com.example.datastoreapp.data.model

data class UserSettings(
    val isDarkTheme: Boolean = false,
    val language: String = "en",
    val fontSize: Int = 14,
    val notificationsEnabled: Boolean = true
)
```
```kotlin
// data/model/AuthData.kt
package com.example.datastoreapp.data.model

data class AuthData(
    val accessToken: String = "",
    val refreshToken: String = "",
    val userId: String = "",
    val isLoggedIn: Boolean = false,
    val tokenExpiryTime: Long = 0L
)
```
```kotlin
// data/model/AppConfig.kt
package com.example.datastoreapp.data.model

data class AppConfig(
    val isOnboardingCompleted: Boolean = false,
    val appVersion: String = "",
    val lastSyncTime: Long = 0L,
    val isFirstLaunch: Boolean = true
)
```

### Step 4: Create Wrapper Classes

#### UserSettings DataStore Wrapper
```kotlin
// data/local/datastore/UserSettingsDataStore.kt
package com.example.datastoreapp.data.local.datastore

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
    
    private object PreferenceKeys {
        val THEME = booleanPreferencesKey("is_dark_theme")
        val LANGUAGE = stringPreferencesKey("language")
        val FONT_SIZE = intPreferencesKey("font_size")
        val NOTIFICATIONS = booleanPreferencesKey("notifications_enabled")
    }
    
    /**
     * Get all user settings as a single Flow
     */
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
                isDarkTheme = preferences[PreferenceKeys.THEME] ?: false,
                language = preferences[PreferenceKeys.LANGUAGE] ?: "en",
                fontSize = preferences[PreferenceKeys.FONT_SIZE] ?: 14,
                notificationsEnabled = preferences[PreferenceKeys.NOTIFICATIONS] ?: true
            )
        }
    
    /**
     * Get individual properties
     */
    val isDarkTheme: Flow<Boolean> = dataStore.data
        .catch { exception ->
            if (exception is IOException) {
                emit(emptyPreferences())
            } else {
                throw exception
            }
        }
        .map { preferences ->
            preferences[PreferenceKeys.THEME] ?: false
        }
    
    val language: Flow<String> = dataStore.data
        .catch { exception ->
            if (exception is IOException) {
                emit(emptyPreferences())
            } else {
                throw exception
            }
        }
        .map { preferences ->
            preferences[PreferenceKeys.LANGUAGE] ?: "en"
        }
    
    val fontSize: Flow<Int> = dataStore.data
        .catch { exception ->
            if (exception is IOException) {
                emit(emptyPreferences())
            } else {
                throw exception
            }
        }
        .map { preferences ->
            preferences[PreferenceKeys.FONT_SIZE] ?: 14
        }
    
    /**
     * Update theme
     */
    suspend fun updateTheme(isDark: Boolean) {
        dataStore.edit { preferences ->
            preferences[PreferenceKeys.THEME] = isDark
        }
    }
    
    /**
     * Update language
     */
    suspend fun updateLanguage(language: String) {
        dataStore.edit { preferences ->
            preferences[PreferenceKeys.LANGUAGE] = language
        }
    }
    
    /**
     * Update font size
     */
    suspend fun updateFontSize(size: Int) {
        dataStore.edit { preferences ->
            preferences[PreferenceKeys.FONT_SIZE] = size
        }
    }
    
    /**
     * Update notifications
     */
    suspend fun updateNotifications(enabled: Boolean) {
        dataStore.edit { preferences ->
            preferences[PreferenceKeys.NOTIFICATIONS] = enabled
        }
    }
    
    /**
     * Batch update all settings
     */
    suspend fun updateAllSettings(settings: UserSettings) {
        dataStore.edit { preferences ->
            preferences[PreferenceKeys.THEME] = settings.isDarkTheme
            preferences[PreferenceKeys.LANGUAGE] = settings.language
            preferences[PreferenceKeys.FONT_SIZE] = settings.fontSize
            preferences[PreferenceKeys.NOTIFICATIONS] = settings.notificationsEnabled
        }
    }
    
    /**
     * Clear all settings
     */
    suspend fun clearAllSettings() {
        dataStore.edit { preferences ->
            preferences.clear()
        }
    }
}
```

#### Auth DataStore Wrapper
```kotlin
// data/local/datastore/AuthDataStore.kt
package com.example.datastoreapp.data.local.datastore

import androidx.datastore.core.DataStore
import androidx.datastore.preferences.core.Preferences
import androidx.datastore.preferences.core.booleanPreferencesKey
import androidx.datastore.preferences.core.edit
import androidx.datastore.preferences.core.emptyPreferences
import androidx.datastore.preferences.core.longPreferencesKey
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
    
    private object PreferenceKeys {
        val ACCESS_TOKEN = stringPreferencesKey("access_token")
        val REFRESH_TOKEN = stringPreferencesKey("refresh_token")
        val USER_ID = stringPreferencesKey("user_id")
        val IS_LOGGED_IN = booleanPreferencesKey("is_logged_in")
        val TOKEN_EXPIRY_TIME = longPreferencesKey("token_expiry_time")
    }
    
    /**
     * Get all auth data
     */
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
                accessToken = preferences[PreferenceKeys.ACCESS_TOKEN] ?: "",
                refreshToken = preferences[PreferenceKeys.REFRESH_TOKEN] ?: "",
                userId = preferences[PreferenceKeys.USER_ID] ?: "",
                isLoggedIn = preferences[PreferenceKeys.IS_LOGGED_IN] ?: false,
                tokenExpiryTime = preferences[PreferenceKeys.TOKEN_EXPIRY_TIME] ?: 0L
            )
        }
    
    /**
     * Check if user is logged in
     */
    val isLoggedIn: Flow<Boolean> = dataStore.data
        .catch { exception ->
            if (exception is IOException) {
                emit(emptyPreferences())
            } else {
                throw exception
            }
        }
        .map { preferences ->
            preferences[PreferenceKeys.IS_LOGGED_IN] ?: false
        }
    
    /**
     * Get access token
     */
    val accessToken: Flow<String> = dataStore.data
        .catch { exception ->
            if (exception is IOException) {
                emit(emptyPreferences())
            } else {
                throw exception
            }
        }
        .map { preferences ->
            preferences[PreferenceKeys.ACCESS_TOKEN] ?: ""
        }
    
    /**
     * Save auth tokens
     */
    suspend fun saveAuthTokens(
        accessToken: String,
        refreshToken: String,
        userId: String,
        expiryTime: Long
    ) {
        dataStore.edit { preferences ->
            preferences[PreferenceKeys.ACCESS_TOKEN] = accessToken
            preferences[PreferenceKeys.REFRESH_TOKEN] = refreshToken
            preferences[PreferenceKeys.USER_ID] = userId
            preferences[PreferenceKeys.IS_LOGGED_IN] = true
            preferences[PreferenceKeys.TOKEN_EXPIRY_TIME] = expiryTime
        }
    }
    
    /**
     * Update access token only
     */
    suspend fun updateAccessToken(token: String, expiryTime: Long) {
        dataStore.edit { preferences ->
            preferences[PreferenceKeys.ACCESS_TOKEN] = token
            preferences[PreferenceKeys.TOKEN_EXPIRY_TIME] = expiryTime
        }
    }
    
    /**
     * Clear auth data (logout)
     */
    suspend fun clearAuthData() {
        dataStore.edit { preferences ->
            preferences.clear()
        }
    }
    
    /**
     * Update login status
     */
    suspend fun setLoggedIn(isLoggedIn: Boolean) {
        dataStore.edit { preferences ->
            preferences[PreferenceKeys.IS_LOGGED_IN] = isLoggedIn
        }
    }
}
```

#### AppConfig DataStore Wrapper
```kotlin
// data/local/datastore/AppConfigDataStore.kt
package com.example.datastoreapp.data.local.datastore

import androidx.datastore.core.DataStore
import androidx.datastore.preferences.core.Preferences
import androidx.datastore.preferences.core.booleanPreferencesKey
import androidx.datastore.preferences.core.edit
import androidx.datastore.preferences.core.emptyPreferences
import androidx.datastore.preferences.core.longPreferencesKey
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
    
    private object PreferenceKeys {
        val ONBOARDING_COMPLETED = booleanPreferencesKey("onboarding_completed")
        val APP_VERSION = stringPreferencesKey("app_version")
        val LAST_SYNC_TIME = longPreferencesKey("last_sync_time")
        val IS_FIRST_LAUNCH = booleanPreferencesKey("is_first_launch")
    }
    
    /**
     * Get all app config
     */
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
                isOnboardingCompleted = preferences[PreferenceKeys.ONBOARDING_COMPLETED] ?: false,
                appVersion = preferences[PreferenceKeys.APP_VERSION] ?: "",
                lastSyncTime = preferences[PreferenceKeys.LAST_SYNC_TIME] ?: 0L,
                isFirstLaunch = preferences[PreferenceKeys.IS_FIRST_LAUNCH] ?: true
            )
        }
    
    /**
     * Check if onboarding is completed
     */
    val isOnboardingCompleted: Flow<Boolean> = dataStore.data
        .catch { exception ->
            if (exception is IOException) {
                emit(emptyPreferences())
            } else {
                throw exception
            }
        }
        .map { preferences ->
            preferences[PreferenceKeys.ONBOARDING_COMPLETED] ?: false
        }
    
    /**
     * Check if first launch
     */
    val isFirstLaunch: Flow<Boolean> = dataStore.data
        .catch { exception ->
            if (exception is IOException) {
                emit(emptyPreferences())
            } else {
                throw exception
            }
        }
        .map { preferences ->
            preferences[PreferenceKeys.IS_FIRST_LAUNCH] ?: true
        }
    
    /**
     * Mark onboarding as completed
     */
    suspend fun setOnboardingCompleted() {
        dataStore.edit { preferences ->
            preferences[PreferenceKeys.ONBOARDING_COMPLETED] = true
        }
    }
    
    /**
     * Update app version
     */
    suspend fun updateAppVersion(version: String) {
        dataStore.edit { preferences ->
            preferences[PreferenceKeys.APP_VERSION] = version
        }
    }
    
    /**
     * Update last sync time
     */
    suspend fun updateLastSyncTime(time: Long) {
        dataStore.edit { preferences ->
            preferences[PreferenceKeys.LAST_SYNC_TIME] = time
        }
    }
    
    /**
     * Mark app as not first launch
     */
    suspend fun setNotFirstLaunch() {
        dataStore.edit { preferences ->
            preferences[PreferenceKeys.IS_FIRST_LAUNCH] = false
        }
    }
    
    /**
     * Clear all config
     */
    suspend fun clearConfig() {
        dataStore.edit { preferences ->
            preferences.clear()
        }
    }
}
```

### Step 5: Application Class
```kotlin
// MyApplication.kt
package com.example.datastoreapp

import android.app.Application
import dagger.hilt.android.HiltAndroidApp

@HiltAndroidApp
class MyApplication : Application()
```

**AndroidManifest.xml:**
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

---

## Method 2: Using Extension Property (Without Factory)

This method is simpler and requires less boilerplate code.

### Step 1: Create Extension Properties
```kotlin
// di/DataStoreExtensions.kt
package com.example.datastoreapp.di

import android.content.Context
import androidx.datastore.core.DataStore
import androidx.datastore.preferences.core.Preferences
import androidx.datastore.preferences.preferencesDataStore

/**
 * Extension properties with delegation
 * These are automatically thread-safe and singleton
 */

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

### Step 2: Create Simplified Module
```kotlin
// di/DataStoreModule.kt (Method 2)
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

**Note:** All other code (Qualifiers, Data Models, Wrapper Classes, Application Class) remains exactly the same as Method 1!

---

## Usage Examples

### Example 1: Use in ViewModel
```kotlin
// presentation/viewmodel/SettingsViewModel.kt
package com.example.datastoreapp.presentation.viewmodel

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.example.datastoreapp.data.local.datastore.UserSettingsDataStoreWrapper
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
    
    // Collect settings as StateFlow
    val userSettings: StateFlow<UserSettings> = userSettingsDataStore.userSettings
        .stateIn(
            scope = viewModelScope,
            started = SharingStarted.WhileSubscribed(5000),
            initialValue = UserSettings()
        )
    
    // Individual property
    val isDarkTheme: StateFlow<Boolean> = userSettingsDataStore.isDarkTheme
        .stateIn(
            scope = viewModelScope,
            started = SharingStarted.WhileSubscribed(5000),
            initialValue = false
        )
    
    fun updateTheme(isDark: Boolean) {
        viewModelScope.launch {
            userSettingsDataStore.updateTheme(isDark)
        }
    }
    
    fun updateLanguage(language: String) {
        viewModelScope.launch {
            userSettingsDataStore.updateLanguage(language)
        }
    }
    
    fun updateFontSize(size: Int) {
        viewModelScope.launch {
            userSettingsDataStore.updateFontSize(size)
        }
    }
    
    fun resetSettings() {
        viewModelScope.launch {
            userSettingsDataStore.clearAllSettings()
        }
    }
}
```

### Example 2: Use in Activity
```kotlin
// presentation/MainActivity.kt
package com.example.datastoreapp.presentation

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.viewModels
import androidx.lifecycle.Lifecycle
import androidx.lifecycle.lifecycleScope
import androidx.lifecycle.repeatOnLifecycle
import com.example.datastoreapp.presentation.viewmodel.SettingsViewModel
import dagger.hilt.android.AndroidEntryPoint
import kotlinx.coroutines.launch

@AndroidEntryPoint
class MainActivity : ComponentActivity() {
    
    private val settingsViewModel: SettingsViewModel by viewModels()
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // Collect settings
        lifecycleScope.launch {
            repeatOnLifecycle(Lifecycle.State.STARTED) {
                settingsViewModel.userSettings.collect { settings ->
                    // Update UI based on settings
                    updateUI(settings.isDarkTheme, settings.language)
                }
            }
        }
        
        // Example: Update theme
        // settingsViewModel.updateTheme(true)
    }
    
    private fun updateUI(isDarkTheme: Boolean, language: String) {
        // Your UI update logic
    }
}
```

### Example 3: Direct DataStore Wrapper Usage in Repository
```kotlin
// data/repository/UserRepository.kt
package com.example.datastoreapp.data.repository

import com.example.datastoreapp.data.local.datastore.AuthDataStoreWrapper
import com.example.datastoreapp.data.local.datastore.UserSettingsDataStoreWrapper
import kotlinx.coroutines.flow.first
import javax.inject.Inject
import javax.inject.Singleton

@Singleton
class UserRepository @Inject constructor(
    private val userSettingsDataStore: UserSettingsDataStoreWrapper,
    private val authDataStore: AuthDataStoreWrapper
) {
    
    /**
     * Login user
     */
    suspend fun loginUser(
        accessToken: String,
        refreshToken: String,
        userId: String
    ): Result<Unit> {
        return try {
            val expiryTime = System.currentTimeMillis() + (60 * 60 * 1000) // 1 hour
            
            authDataStore.saveAuthTokens(
                accessToken = accessToken,
                refreshToken = refreshToken,
                userId = userId,
                expiryTime = expiryTime
            )
            
            Result.success(Unit)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
    
    /**
     * Logout user
     */
    suspend fun logoutUser(): Result<Unit> {
        return try {
            authDataStore.clearAuthData()
            userSettingsDataStore.clearAllSettings()
            Result.success(Unit)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
    
    /**
     * Check if user is logged in
     */
    suspend fun isUserLoggedIn(): Boolean {
        return authDataStore.isLoggedIn.first()
    }
    
    /**
     * Get user's preferred language
     */
    suspend fun getUserLanguage(): String {
        return userSettingsDataStore.language.first()
    }
    
    /**
     * Update user preferences
     */
    suspend fun updateUserPreferences(
        isDarkTheme: Boolean,
        language: String,
        fontSize: Int
    ): Result<Unit> {
        return try {
            userSettingsDataStore.updateTheme(isDarkTheme)
            userSettingsDataStore.updateLanguage(language)
            userSettingsDataStore.updateFontSize(fontSize)
            Result.success(Unit)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}
```

### Example 4: Combine Multiple DataStores
```kotlin
// domain/usecase/GetAppStateUseCase.kt
package com.example.datastoreapp.domain.usecase

import com.example.datastoreapp.data.local.datastore.AppConfigDataStoreWrapper
import com.example.datastoreapp.data.local.datastore.AuthDataStoreWrapper
import com.example.datastoreapp.data.local.datastore.UserSettingsDataStoreWrapper
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
    
    /**
     * Combine multiple DataStore flows into one
     */
    operator fun invoke(): Flow<AppState> {
        return combine(
            authDataStore.isLoggedIn,
            userSettingsDataStore.isDarkTheme,
            appConfigDataStore.isOnboardingCompleted,
            userSettingsDataStore.language
        ) { isLoggedIn, isDarkTheme, isOnboardingCompleted, language ->
            AppState(
                isLoggedIn = isLoggedIn,
                isDarkTheme = isDarkTheme,
                isOnboardingCompleted = isOnboardingCompleted,
                language = language
            )
        }
    }
}
```

---

## Testing

### Example Unit Test
```kotlin
// test/data/local/datastore/UserSettingsDataStoreTest.kt
package com.example.datastoreapp.data.local.datastore

import android.content.Context
import androidx.datastore.core.DataStore
import androidx.datastore.preferences.core.PreferenceDataStoreFactory
import androidx.datastore.preferences.core.Preferences
import androidx.datastore.preferences.preferencesDataStoreFile
import androidx.test.core.app.ApplicationProvider
import androidx.test.ext.junit.runners.AndroidJUnit4
import kotlinx.coroutines.ExperimentalCoroutinesApi
import kotlinx.coroutines.Job
import kotlinx.coroutines.flow.first
import kotlinx.coroutines.test.TestScope
import kotlinx.coroutines.test.UnconfinedTestDispatcher
import kotlinx.coroutines.test.runTest
import org.junit.After
import org.junit.Assert.*
import org.junit.Before
import org.junit.Test
import org.junit.runner.RunWith

@OptIn(ExperimentalCoroutinesApi::class)
@RunWith(AndroidJUnit4::class)
class UserSettingsDataStoreTest {
    
    private val testContext: Context = ApplicationProvider.getApplicationContext()
    private val testCoroutineDispatcher = UnconfinedTestDispatcher()
    private val testCoroutineScope = TestScope(testCoroutineDispatcher + Job())
    
    private lateinit var dataStore: DataStore<Preferences>
    private lateinit var userSettingsDataStore: UserSettingsDataStoreWrapper
    
    @Before
    fun setup() {
        dataStore = PreferenceDataStoreFactory.create(
            scope = testCoroutineScope,
            produceFile = { testContext.preferencesDataStoreFile("test_user_settings") }
        )
        userSettingsDataStore = UserSettingsDataStoreWrapper(dataStore)
    }
    
    @After
    fun cleanup() {
        testContext.preferencesDataStoreFile("test_user_settings").delete()
    }
    
    @Test
    fun testDefaultUserSettings() = runTest {
        val settings = userSettingsDataStore.userSettings.first()
        
        assertFalse(settings.isDarkTheme)
        assertEquals("en", settings.language)
        assertEquals(14, settings.fontSize)
        assertTrue(settings.notificationsEnabled)
    }
    
    @Test
    fun testUpdateTheme() = runTest {
        userSettingsDataStore.updateTheme(true)
        
        val isDark = userSettingsDataStore.isDarkTheme.first()
        assertTrue(isDark)
    }
    
    @Test
    fun testUpdateLanguage() = runTest {
        userSettingsDataStore.updateLanguage("bn")
        
        val language = userSettingsDataStore.language.first()
        assertEquals("bn", language)
    }
    
    @Test
    fun testUpdateFontSize() = runTest {
        userSettingsDataStore.updateFontSize(18)
        
        val fontSize = userSettingsDataStore.fontSize.first()
        assertEquals(18, fontSize)
    }
    
    @Test
    fun testUpdateAllSettings() = runTest {
        val newSettings = UserSettings(
            isDarkTheme = true,
            language = "bn",
            fontSize = 18,
            notificationsEnabled = false
        )
        
        userSettingsDataStore.updateAllSettings(newSettings)
        
        val settings = userSettingsDataStore.userSettings.first()
        assertEquals(newSettings, settings)
    }
    
    @Test
    fun testClearAllSettings() = runTest {
        // First set some values
        userSettingsDataStore.updateTheme(true)
        userSettingsDataStore.updateLanguage("bn")
        
        // Then clear
        userSettingsDataStore.clearAllSettings()
        
        // Verify defaults are restored
        val settings = userSettingsDataStore.userSettings.first()
        assertFalse(settings.isDarkTheme)
        assertEquals("en", settings.language)
    }
}
```

---

## Comparison

### Method 1: PreferenceDataStoreFactory

**Code Example:**
```kotlin
@Provides
@Singleton
fun provideDataStore(@ApplicationContext context: Context): DataStore<Preferences> {
    return PreferenceDataStoreFactory.create(
        corruptionHandler = ReplaceFileCorruptionHandler { emptyPreferences() },
        scope = CoroutineScope(Dispatchers.IO + SupervisorJob()),
        produceFile = { context.preferencesDataStoreFile("settings") }
    )
}
```

**Advantages:**
- ✅ Full control over configuration
- ✅ Custom corruption handler
- ✅ Custom coroutine scope
- ✅ Advanced error handling
- ✅ Migration support
- ✅ Multi-process support available

**Disadvantages:**
- ❌ More boilerplate code
- ❌ Manual scope management
- ❌ More complex setup

**Best For:**
- Production apps with complex requirements
- Apps needing custom error handling
- Apps requiring data migrations
- Multi-process applications

---

### Method 2: Extension Property (Delegation)

**Code Example:**
```kotlin
val Context.dataStore by preferencesDataStore(name = "settings")

@Provides
@Singleton
fun provideDataStore(@ApplicationContext context: Context): DataStore<Preferences> {
    return context.dataStore
}
```

**Advantages:**
- ✅ Minimal boilerplate
- ✅ Automatic scope management
- ✅ Thread-safe by default
- ✅ Clean and simple
- ✅ Quick setup
- ✅ Singleton automatically

**Disadvantages:**
- ❌ Less customization options
- ❌ Default configurations only
- ❌ No custom corruption handler
- ❌ No custom scope

**Best For:**
- Simple to medium complexity apps
- Standard use cases
- Rapid prototyping
- Learning DataStore
- Projects prioritizing simplicity

---

## Best Practices

### 1. Separation of Concerns

✅ **Do:**
```kotlin
// Separate DataStores for different domains
UserSettingsDataStore  // UI preferences
AuthDataStore          // Authentication data
AppConfigDataStore     // App configuration
```

❌ **Don't:**
```kotlin
// Mixing all data in one DataStore
SingleDataStore  // Everything together (Bad!)
```

### 2. Use Wrapper Classes

✅ **Do:**
```kotlin
@Singleton
class UserSettingsDataStoreWrapper @Inject constructor(
    @UserSettingsDataStore private val dataStore: DataStore<Preferences>
) {
    // All CRUD operations here
}
```

**Benefits:**
- Encapsulation
- Type safety
- Easy testing
- Reusability

### 3. Error Handling

✅ **Do:**
```kotlin
val userSettings: Flow<UserSettings> = dataStore.data
    .catch { exception ->
        if (exception is IOException) {
            emit(emptyPreferences())
        } else {
            throw exception
        }
    }
    .map { /* mapping */ }
```

### 4. Use Flow for Reactive Updates

✅ **Do:**
```kotlin
// Reactive data with Flow
val isDarkTheme: Flow<Boolean> = dataStore.data
    .map { it[THEME_KEY] ?: false }

// In ViewModel
val theme = dataStore.isDarkTheme.stateIn(...)
```

### 5. Qualifiers for Multiple DataStores

✅ **Do:**
```kotlin
@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class UserSettingsDataStore

@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class AuthDataStore
```

### 6. Avoid Blocking Calls

❌ **Don't:**
```kotlin
// Don't use runBlocking in production
val settings = runBlocking {
    dataStore.data.first()
}
```

✅ **Do:**
```kotlin
// Use coroutines properly
viewModelScope.launch {
    val settings = dataStore.data.first()
}
```

### 7. Data Models

✅ **Do:**
```kotlin
// Use data classes for type safety
data class UserSettings(
    val isDarkTheme: Boolean = false,
    val language: String = "en"
)
```

### 8. Testing

✅ **Do:**
```kotlin
// Use TestScope and test DataStore
@Before
fun setup() {
    dataStore = PreferenceDataStoreFactory.create(
        scope = testCoroutineScope,
        produceFile = { testContext.preferencesDataStoreFile("test") }
    )
}

@After
fun cleanup() {
    testContext.preferencesDataStoreFile("test").delete()
}
```

---

## Architecture Flow
```
┌─────────────────────────────────────────────────────────────┐
│                      Hilt Module (DI)                       │
│                 @InstallIn(SingletonComponent)              │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           │ Provides with @Qualifier
                           ▼
┌─────────────────────────────────────────────────────────────┐
│               DataStore<Preferences> Instances              │
│  @UserSettingsDataStore, @AuthDataStore, @AppConfigDataStore│
└──────────────────────────┬──────────────────────────────────┘
                           │
                           │ Injected into
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                    Wrapper Classes                          │
│  UserSettingsDataStoreWrapper, AuthDataStoreWrapper, etc.   │
│              (CRUD Operations + Flow Mapping)               │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           │ Used in
                           ▼
┌─────────────────────────────────────────────────────────────┐
│              Repository / ViewModel / UseCase               │
│                   (Business Logic Layer)                    │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           │ Consumed by
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                 UI Layer (Activity/Fragment)                │
│                  Collect Flow as StateFlow                  │
└─────────────────────────────────────────────────────────────┘
```

---

## Key Takeaways

1. **Multiple DataStores** separate concerns and improve maintainability
2. **Qualifiers** allow Hilt to distinguish between different DataStore instances
3. **Wrapper Classes** encapsulate DataStore operations and provide clean API
4. **Flow API** enables reactive data updates
5. **Method 1 (Factory)** provides more control, **Method 2 (Extension)** is simpler
6. Both methods can coexist in the same project
7. Always handle IOException with proper error recovery
8. Use StateFlow in ViewModels for UI consumption
9. Write tests for DataStore operations
10. Follow Clean Architecture principles

---

## Common Issues and Solutions

### Issue 1: Multiple DataStore Instances

**Problem:**
```kotlin
// Ambiguous DataStore injection
class MyClass @Inject constructor(
    private val dataStore: DataStore<Preferences> // Which one?
)
```

**Solution:**
```kotlin
// Use qualifiers
class MyClass @Inject constructor(
    @UserSettingsDataStore private val dataStore: DataStore<Preferences>
)
```

### Issue 2: DataStore File Already Exists

**Problem:**
```
java.lang.IllegalStateException: DataStore file already exists
```

**Solution:**
```kotlin
// Use unique names for each DataStore
val Context.userSettings by preferencesDataStore("user_settings")
val Context.authData by preferencesDataStore("auth_data") // Different name
```

### Issue 3: Cannot Access From Multiple Processes

**Problem:**
```
Can't use DataStore from multiple processes
```

**Solution:**
```kotlin
// Use multiProcessDataStore for multi-process apps
val Context.dataStore by multiProcessPreferencesDataStore("settings")
```

---

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

---

## License

This tutorial is available under the MIT License.

---

## Resources

- [Official DataStore Documentation](https://developer.android.com/topic/libraries/architecture/datastore)
- [Hilt Documentation](https://dagger.dev/hilt/)
- [Kotlin Coroutines Guide](https://kotlinlang.org/docs/coroutines-guide.html)
- [Flow Documentation](https://kotlinlang.org/docs/flow.html)

---

## Author

Created as a comprehensive guide for Android developers working with DataStore and Hilt.

---

**Happy Coding! 🚀**
