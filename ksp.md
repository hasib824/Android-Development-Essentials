# KSP (Kotlin Symbol Processing) - সম্পূর্ণ Bangla Tutorial

## 📚 Table of Contents
1. [KSP কি?](#ksp-কি)
2. [কেন KSP দরকার?](#কেন-ksp-দরকার)
3. [KSP কিভাবে কাজ করে?](#ksp-কিভাবে-কাজ-করে)
4. [Setup Guide](#setup-guide)
5. [Real World Examples](#real-world-examples)
6. [KAPT vs KSP](#kapt-vs-ksp)
7. [Common Use Cases](#common-use-cases)
8. [Troubleshooting](#troubleshooting)

---

## KSP কি?

**KSP = Kotlin Symbol Processing**

KSP হলো একটা **code generation tool** যা আপনার জন্য **automatic code লিখে দেয়** compile time এ।

### 🎯 সহজ উদাহরণ:
```kotlin
// আপনি এটুকু লিখবেন:
@Entity
data class User(val name: String)

// KSP automatic 500+ lines code generate করবে:
// - Database queries
// - Type converters
// - Error handling
// - SQL statements
// এবং আরও অনেক কিছু!
```

### 💡 মূল Concept:
```
আপনার Code (Annotations সহ)
        ↓
    KSP Process
        ↓
Generated Code (Automatic)
        ↓
    Final App Build
```

---

## কেন KSP দরকার?

### 🤔 সমস্যা: Manual Code লিখতে হতো
```kotlin
// ❌ আগে manually সব লিখতে হতো (500+ lines!)

@Entity
data class User(val id: String, val name: String)

// এই সব manually লিখতে হতো:
class UserDao_Impl : UserDao {
    private val __db: RoomDatabase
    
    override fun insertUser(user: User) {
        __db.assertNotSuspendingTransaction()
        __db.beginTransaction()
        try {
            val statement = __db.compileStatement(
                "INSERT INTO User (id, name) VALUES (?, ?)"
            )
            statement.bindString(1, user.id)
            statement.bindString(2, user.name)
            statement.executeInsert()
            __db.setTransactionSuccessful()
        } finally {
            __db.endTransaction()
        }
    }
    // ... আরও 400+ lines code
}

// 😱 এভাবে প্রতিটা Entity এর জন্য লিখতে হতো!
```

### ✅ সমাধান: KSP Automatic Code Generate করে
```kotlin
// ✅ এখন শুধু এটুকু লিখলেই হয়:

@Entity
data class User(val id: String, val name: String)

@Dao
interface UserDao {
    @Insert
    suspend fun insertUser(user: User)
}

// ✨ KSP বাকি সব automatic করে দেয়!
// ✨ Build time এ 500+ lines code generate হয়!
// ✨ আপনাকে কিছু লিখতে হয় না!
```

### 🎁 KSP এর সুবিধা:

| সুবিধা | বর্ণনা |
|--------|---------|
| ⚡ **Fast** | KAPT থেকে **2x দ্রুত** |
| 💾 **Memory Efficient** | কম RAM use করে |
| 🎯 **Kotlin Native** | Pure Kotlin API |
| 🐛 **Better Errors** | Clear error messages |
| 📦 **Less Code** | Boilerplate automatic |
| 🔮 **Future-proof** | Google officially support করছে |

---

## KSP কিভাবে কাজ করে?

### 📊 Step-by-Step Flow:
```
┌─────────────────────────────────────────────────────┐
│                KSP Workflow                         │
└─────────────────────────────────────────────────────┘

Step 1: আপনি Code লিখেন
┌──────────────────────────────────┐
│  @Entity                         │
│  data class User(val name: String)│
│                                  │
│  @Dao                            │
│  interface UserDao {             │
│      @Insert                     │
│      suspend fun insert(user: User)│
│  }                               │
└──────────────────────────────────┘
              ↓
Step 2: Build/Compile শুরু হয়
┌──────────────────────────────────┐
│  ./gradlew build                 │
│  Kotlin Compiler starts          │
└──────────────────────────────────┘
              ↓
Step 3: KSP Processor কাজ করে
┌──────────────────────────────────┐
│  1. Annotations scan করে         │
│  2. Classes analyze করে          │
│  3. Code generate করে            │
└──────────────────────────────────┘
              ↓
Step 4: Generated Code তৈরি হয়
┌──────────────────────────────────┐
│  build/generated/ksp/            │
│  ├── UserDao_Impl.kt             │
│  ├── User_Impl.kt                │
│  └── AppDatabase_Impl.kt         │
└──────────────────────────────────┘
              ↓
Step 5: Final Build Complete
┌──────────────────────────────────┐
│  Your Code + Generated Code      │
│  = Working App! 🎉               │
└──────────────────────────────────┘
```

### 🔍 Real Example - Room Database:
```kotlin
// আপনার Code (10 lines):
@Entity(tableName = "users")
data class User(
    @PrimaryKey val id: String,
    val name: String,
    val email: String,
    val age: Int
)

@Dao
interface UserDao {
    @Insert
    suspend fun insertUser(user: User)
    
    @Query("SELECT * FROM users WHERE age > :minAge")
    suspend fun getUsersAboveAge(minAge: Int): List<User>
    
    @Update
    suspend fun updateUser(user: User)
    
    @Delete
    suspend fun deleteUser(user: User)
}

// ↓↓↓ KSP Automatic এই সব generate করে (500+ lines) ↓↓↓

// UserDao_Impl.kt (Generated by KSP):
class UserDao_Impl(private val __db: RoomDatabase) : UserDao {
    
    private val __insertionAdapter = EntityInsertionAdapter<User>(__db) {
        override fun createQuery(): String {
            return "INSERT OR ABORT INTO `users` " +
                   "(`id`,`name`,`email`,`age`) VALUES (?,?,?,?)"
        }
        
        override fun bind(stmt: SQLiteStatement, entity: User) {
            stmt.bindString(1, entity.id)
            stmt.bindString(2, entity.name)
            stmt.bindString(3, entity.email)
            stmt.bindLong(4, entity.age.toLong())
        }
    }
    
    private val __updateAdapter = EntityDeletionOrUpdateAdapter<User>(__db) {
        override fun createQuery(): String {
            return "UPDATE OR ABORT `users` SET " +
                   "`id` = ?,`name` = ?,`email` = ?,`age` = ? " +
                   "WHERE `id` = ?"
        }
        
        override fun bind(stmt: SQLiteStatement, entity: User) {
            stmt.bindString(1, entity.id)
            stmt.bindString(2, entity.name)
            stmt.bindString(3, entity.email)
            stmt.bindLong(4, entity.age.toLong())
            stmt.bindString(5, entity.id)
        }
    }
    
    override suspend fun insertUser(user: User) {
        __db.assertNotSuspendingTransaction()
        __db.beginTransaction()
        try {
            __insertionAdapter.insert(user)
            __db.setTransactionSuccessful()
        } finally {
            __db.endTransaction()
        }
    }
    
    override suspend fun getUsersAboveAge(minAge: Int): List<User> {
        val _sql = "SELECT * FROM users WHERE age > ?"
        val _statement = RoomSQLiteQuery.acquire(_sql, 1)
        
        _statement.bindLong(1, minAge.toLong())
        
        __db.assertNotSuspendingTransaction()
        val _cursor = DBUtil.query(__db, _statement, false, null)
        
        try {
            val _idIndex = CursorUtil.getColumnIndexOrThrow(_cursor, "id")
            val _nameIndex = CursorUtil.getColumnIndexOrThrow(_cursor, "name")
            val _emailIndex = CursorUtil.getColumnIndexOrThrow(_cursor, "email")
            val _ageIndex = CursorUtil.getColumnIndexOrThrow(_cursor, "age")
            
            val _result = mutableListOf<User>()
            while (_cursor.moveToNext()) {
                val _item = User(
                    id = _cursor.getString(_idIndex),
                    name = _cursor.getString(_nameIndex),
                    email = _cursor.getString(_emailIndex),
                    age = _cursor.getInt(_ageIndex)
                )
                _result.add(_item)
            }
            return _result
        } finally {
            _cursor.close()
            _statement.release()
        }
    }
    
    // ... আরও 300+ lines generated code
}

// ✨ এই পুরো complex code KSP automatic লিখেছে!
// ✨ আপনি শুধু @Entity, @Dao, @Query লিখেছেন!
```

---

## Setup Guide

### 📋 Prerequisites:

- ✅ Android Studio (latest version)
- ✅ Kotlin 1.9.22+ (or latest)
- ✅ Gradle 8.0+

### 🛠️ Step 1: versions.toml Setup
```toml
# gradle/libs.versions.toml

[versions]
kotlin = "1.9.22"
ksp = "1.9.22-1.0.17"  # ⚠️ Must match Kotlin version!
room = "2.6.1"
hilt = "2.50"

[plugins]
# KSP Plugin
ksp = { id = "com.google.devtools.ksp", version.ref = "ksp" }

[libraries]
# Room
room-runtime = { group = "androidx.room", name = "room-runtime", version.ref = "room" }
room-compiler = { group = "androidx.room", name = "room-compiler", version.ref = "room" }
room-ktx = { group = "androidx.room", name = "room-ktx", version.ref = "room" }

# Hilt
hilt-android = { group = "com.google.dagger", name = "hilt-android", version.ref = "hilt" }
hilt-compiler = { group = "com.google.dagger", name = "hilt-compiler", version.ref = "hilt" }
```

### 🛠️ Step 2: build.gradle.kts (App Module)
```kotlin
// build.gradle.kts (Module :app)

plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.kotlin.android)
    alias(libs.plugins.ksp)  // ✅ KSP plugin apply করুন
}

android {
    namespace = "com.hasib.myapp"
    compileSdk = 34
    
    defaultConfig {
        applicationId = "com.hasib.myapp"
        minSdk = 24
        targetSdk = 34
        versionCode = 1
        versionName = "1.0"
    }
    
    buildFeatures {
        compose = true
    }
    
    composeOptions {
        kotlinCompilerExtensionVersion = "1.5.8"
    }
}

dependencies {
    // Room Database
    implementation(libs.room.runtime)
    implementation(libs.room.ktx)
    ksp(libs.room.compiler)  // ✅ ksp() use করুন
    
    // Hilt Dependency Injection
    implementation(libs.hilt.android)
    ksp(libs.hilt.compiler)  // ✅ ksp() use করুন
}
```

### ⚠️ Important Notes:

#### KSP Version Matching:
```toml
# ✅ CORRECT - Versions match
kotlin = "1.9.22"
ksp = "1.9.22-1.0.17"  # First part matches Kotlin version

# ❌ WRONG - Versions don't match
kotlin = "1.9.22"
ksp = "1.9.20-1.0.14"  # Mismatch! Will cause errors
```

#### Finding Compatible KSP Version:
```
🔗 Official KSP Releases:
https://github.com/google/ksp/releases

Rule: KSP version = <Kotlin Version>-<KSP Release>

Examples:
Kotlin 1.9.22 → ksp = "1.9.22-1.0.17"
Kotlin 1.9.24 → ksp = "1.9.24-1.0.20"
Kotlin 2.0.0  → ksp = "2.0.0-1.0.21"
```

---

## Real World Examples

### 🗄️ Example 1: Room Database (Complete Setup)

#### Step 1: Entity তৈরি করুন
```kotlin
// data/local/entity/UserEntity.kt

import androidx.room.Entity
import androidx.room.PrimaryKey

@Entity(tableName = "users")
data class UserEntity(
    @PrimaryKey
    val id: String,
    
    val name: String,
    val email: String,
    val age: Int,
    val createdAt: Long = System.currentTimeMillis()
)
```

#### Step 2: DAO তৈরি করুন
```kotlin
// data/local/dao/UserDao.kt

import androidx.room.*
import kotlinx.coroutines.flow.Flow

@Dao
interface UserDao {
    
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertUser(user: UserEntity)
    
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertUsers(users: List<UserEntity>)
    
    @Query("SELECT * FROM users")
    fun getAllUsers(): Flow<List<UserEntity>>
    
    @Query("SELECT * FROM users WHERE id = :userId")
    suspend fun getUserById(userId: String): UserEntity?
    
    @Query("SELECT * FROM users WHERE age > :minAge ORDER BY age ASC")
    suspend fun getUsersAboveAge(minAge: Int): List<UserEntity>
    
    @Update
    suspend fun updateUser(user: UserEntity)
    
    @Delete
    suspend fun deleteUser(user: UserEntity)
    
    @Query("DELETE FROM users")
    suspend fun deleteAllUsers()
    
    @Query("SELECT COUNT(*) FROM users")
    suspend fun getUserCount(): Int
}
```

#### Step 3: Database তৈরি করুন
```kotlin
// data/local/AppDatabase.kt

import androidx.room.Database
import androidx.room.RoomDatabase

@Database(
    entities = [UserEntity::class],
    version = 1,
    exportSchema = false
)
abstract class AppDatabase : RoomDatabase() {
    abstract fun userDao(): UserDao
}
```

#### Step 4: DatabaseModule (Hilt)
```kotlin
// data/di/DatabaseModule.kt

import android.content.Context
import androidx.room.Room
import dagger.Module
import dagger.Provides
import dagger.hilt.InstallIn
import dagger.hilt.android.qualifiers.ApplicationContext
import dagger.hilt.components.SingletonComponent
import javax.inject.Singleton

@Module
@InstallIn(SingletonComponent::class)
object DatabaseModule {
    
    @Provides
    @Singleton
    fun provideDatabase(
        @ApplicationContext context: Context
    ): AppDatabase {
        return Room.databaseBuilder(
            context,
            AppDatabase::class.java,
            "app_database"
        )
            .fallbackToDestructiveMigration()
            .build()
    }
    
    @Provides
    @Singleton
    fun provideUserDao(database: AppDatabase): UserDao {
        return database.userDao()
    }
}
```

#### 🎉 Result: KSP Generate করবে
```
build/generated/ksp/debug/kotlin/
├── UserEntity_Impl.kt           (Entity implementation)
├── UserDao_Impl.kt              (DAO implementation)
├── AppDatabase_Impl.kt          (Database implementation)
└── UserDao_Impl$insertUser$1.kt (Query implementations)

Total Generated Lines: ~1000+ lines! ✨
```

---

### 💉 Example 2: Hilt Dependency Injection

#### Step 1: Application Class
```kotlin
// MyApplication.kt

import android.app.Application
import dagger.hilt.android.HiltAndroidApp

@HiltAndroidApp  // ✅ KSP will process this
class MyApplication : Application()
```

#### Step 2: Activity
```kotlin
// MainActivity.kt

import androidx.activity.ComponentActivity
import dagger.hilt.android.AndroidEntryPoint

@AndroidEntryPoint  // ✅ KSP will process this
class MainActivity : ComponentActivity() {
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            MyApp()
        }
    }
}
```

#### Step 3: ViewModel
```kotlin
// presentation/viewmodel/UserViewModel.kt

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import dagger.hilt.android.lifecycle.HiltViewModel
import javax.inject.Inject

@HiltViewModel  // ✅ KSP will process this
class UserViewModel @Inject constructor(
    private val userDao: UserDao  // ✅ Auto-injected
) : ViewModel() {
    
    val users = userDao.getAllUsers()
        .stateIn(
            scope = viewModelScope,
            started = SharingStarted.WhileSubscribed(5000),
            initialValue = emptyList()
        )
    
    fun addUser(name: String, email: String, age: Int) {
        viewModelScope.launch {
            val user = UserEntity(
                id = UUID.randomUUID().toString(),
                name = name,
                email = email,
                age = age
            )
            userDao.insertUser(user)
        }
    }
}
```

#### 🎉 Result: KSP Generate করবে
```
build/generated/ksp/debug/kotlin/
├── Hilt_MyApplication.kt
├── MyApplication_GeneratedInjector.kt
├── MainActivity_GeneratedInjector.kt
├── UserViewModel_Factory.kt
├── UserViewModel_HiltModules.kt
└── ... (50+ more files)

Total Generated Lines: ~2000+ lines! ✨
```

---

## KAPT vs KSP

### 📊 Performance Comparison:

| Metric | KAPT | KSP | Improvement |
|--------|------|-----|-------------|
| **Build Time** | 120 seconds | 60 seconds | **2x faster** ⚡ |
| **Memory Usage** | 2.5 GB | 1.5 GB | **40% less** 💾 |
| **API Complexity** | High | Low | **Easier** 🎯 |
| **Error Messages** | Unclear | Clear | **Better** 🐛 |
| **Future Support** | Deprecated | Active | **Maintained** ✅ |

### 🔄 Migration Example:

#### Before (KAPT) - ❌ Old Way:
```kotlin
// build.gradle.kts (OLD)

plugins {
    id("kotlin-kapt")  // ❌ Old plugin
}

dependencies {
    // Room
    implementation("androidx.room:room-runtime:2.6.1")
    kapt("androidx.room:room-compiler:2.6.1")  // ❌ kapt
    
    // Hilt
    implementation("com.google.dagger:hilt-android:2.50")
    kapt("com.google.dagger:hilt-compiler:2.50")  // ❌ kapt
}

// Result:
// ❌ Slow build (2-3 minutes)
// ❌ High memory usage
// ❌ Unclear errors
```

#### After (KSP) - ✅ New Way:
```kotlin
// build.gradle.kts (NEW)

plugins {
    id("com.google.devtools.ksp")  // ✅ New plugin
}

dependencies {
    // Room
    implementation("androidx.room:room-runtime:2.6.1")
    ksp("androidx.room:room-compiler:2.6.1")  // ✅ ksp
    
    // Hilt
    implementation("com.google.dagger:hilt-android:2.50")
    ksp("com.google.dagger:hilt-compiler:2.50")  // ✅ ksp
}

// Result:
// ✅ Fast build (1 minute)
// ✅ Low memory usage
// ✅ Clear errors
```

### 🚀 Migration Steps:
```bash
# Step 1: Update versions.toml
# Add KSP version matching your Kotlin version

# Step 2: Replace plugin
plugins {
    id("kotlin-kapt")  # Remove
    id("com.google.devtools.ksp")  # Add
}

# Step 3: Replace all kapt() with ksp()
dependencies {
    kapt("...")  # Replace all
    ↓
    ksp("...")   # With this
}

# Step 4: Clean and rebuild
./gradlew clean
./gradlew build

# Step 5: Test thoroughly
# Usually migration is smooth! 🎉
```

---

## Common Use Cases

### 1️⃣ Room Database
```kotlin
// Use Case: Local data storage

@Entity
data class Note(
    @PrimaryKey val id: String,
    val title: String,
    val content: String
)

@Dao
interface NoteDao {
    @Insert
    suspend fun insert(note: Note)
    
    @Query("SELECT * FROM Note")
    fun getAll(): Flow<List<Note>>
}

// ✨ KSP generates all database code!
```

### 2️⃣ Hilt Dependency Injection
```kotlin
// Use Case: Managing dependencies

@HiltViewModel
class MyViewModel @Inject constructor(
    private val repository: Repository,
    private val analytics: Analytics
) : ViewModel()

// ✨ KSP generates all injection code!
```

### 3️⃣ Moshi JSON Parsing
```kotlin
// Use Case: API response parsing

@JsonClass(generateAdapter = true)
data class User(
    val id: String,
    val name: String,
    @Json(name = "email_address")
    val email: String
)

// ✨ KSP generates JSON adapter code!
```

### 4️⃣ Glide Image Loading
```kotlin
// Use Case: Image loading with caching

@GlideModule
class MyGlideModule : AppGlideModule()

// Usage:
GlideApp.with(context)
    .load(imageUrl)
    .into(imageView)

// ✨ KSP generates GlideApp class!
```

---

## Troubleshooting

### ❌ Problem 1: KSP Version Mismatch
```kotlin
Error: This version of KSP requires Kotlin 1.9.22 but you have 1.9.20

✅ Solution:
// versions.toml
kotlin = "1.9.22"
ksp = "1.9.22-1.0.17"  // First part must match!
```

### ❌ Problem 2: Generated Code Not Found
```kotlin
Error: Unresolved reference: UserDao_Impl

✅ Solution:
1. Clean project: ./gradlew clean
2. Rebuild: ./gradlew build
3. Invalidate caches: File → Invalidate Caches → Restart
4. Sync Gradle: File → Sync Project with Gradle Files
```

### ❌ Problem 3: Build Slow After KSP
```kotlin
✅ Solution:
// gradle.properties
# Enable Kotlin incremental compilation
kotlin.incremental=true

# Enable KSP incremental processing
ksp.incremental=true

# Enable parallel builds
org.gradle.parallel=true

# Increase memory
org.gradle.jvmargs=-Xmx4096m
```

### ❌ Problem 4: Missing Generated Files
```kotlin
✅ Check these locations:
build/generated/ksp/debug/kotlin/
build/generated/ksp/release/kotlin/

✅ If empty:
1. Verify annotations are correct
2. Check dependencies are added
3. Rebuild project
4. Check Build Output for errors
```

---

## 📝 Cheat Sheet

### Essential Commands:
```bash
# Clean build
./gradlew clean

# Build with KSP
./gradlew build

# Check KSP generated files
ls -la build/generated/ksp/debug/kotlin/

# Build specific variant
./gradlew assembleDebug
./gradlew assembleRelease
```

### Common Annotations:
```kotlin
// Room
@Entity, @PrimaryKey, @ColumnInfo
@Dao, @Insert, @Update, @Delete, @Query
@Database, @TypeConverter

// Hilt
@HiltAndroidApp, @AndroidEntryPoint
@HiltViewModel, @Inject
@Module, @InstallIn, @Provides, @Binds

// Moshi
@JsonClass(generateAdapter = true)
@Json(name = "field_name")
```

### Dependencies Quick Reference:
```kotlin
// Room
ksp("androidx.room:room-compiler:2.6.1")

// Hilt
ksp("com.google.dagger:hilt-compiler:2.50")

// Moshi
ksp("com.squareup.moshi:moshi-kotlin-codegen:1.15.0")

// Glide
ksp("com.github.bumptech.glide:ksp:4.16.0")
```

---

## 🎓 Key Takeaways
```
✅ KSP = Automatic Code Generator
✅ 2x faster than KAPT
✅ Less memory, better errors
✅ Use with Room, Hilt, Moshi, etc.
✅ Add annotation → KSP generates code
✅ No manual boilerplate needed!

Remember:
📌 KSP version must match Kotlin version
📌 Use ksp() not kapt() in dependencies
📌 Generated code in build/generated/ksp/
📌 Always clean and rebuild after changes
```

---

## 🔗 Useful Resources

- [Official KSP Documentation](https://kotlinlang.org/docs/ksp-overview.html)
- [KSP GitHub Releases](https://github.com/google/ksp/releases)
- [Room with KSP](https://developer.android.com/jetpack/androidx/releases/room#ksp)
- [Hilt with KSP](https://developer.android.com/training/dependency-injection/hilt-android)

---

## 📞 Need Help?

যদি কোনো সমস্যা হয়:

1. ✅ Clean and rebuild
2. ✅ Check KSP version matches Kotlin
3. ✅ Verify dependencies are correct
4. ✅ Check generated files exist
5. ✅ Read error messages carefully

Happy Coding! 🚀✨

---

*Last Updated: December 2024*
