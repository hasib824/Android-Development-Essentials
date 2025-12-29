# 📱 Hilt Setup Complete Cheat Sheet
## Retrofit + Room with Hilt Dependency Injection

---

## 🎯 Complete Flow Diagram

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         @HiltAndroidApp                                 │
│                    class MyApp : Application()                          │
└─────────────────────────────────────────────────────────────────────────┘
                                   ↓
                   ┌───────────────┴────────────────┐
                   ↓                                ↓
┌──────────────────────────────────┐  ┌─────────────────────────────────────┐
│      NetworkModule (object)      │  │      DatabaseModule (object)        │
│   @Module                        │  │   @Module                           │
│   @InstallIn(SingletonComponent  │  │   @InstallIn(SingletonComponent     │
│              ::class)            │  │              ::class)               │
├──────────────────────────────────┤  ├─────────────────────────────────────┤
│  Provides:                       │  │  Provides:                          │
│  - Retrofit                      │  │  - AppDatabase                      │
│  - ApiService                    │  │  - UserDao                          │
└──────────────────────────────────┘  └─────────────────────────────────────┘
                   ↓                                ↓
┌──────────────────────────────────┐  ┌─────────────────────────────────────┐
│     ApiService (interface)       │  │  AppDatabase (abstract class)       │
├──────────────────────────────────┤  │  extends RoomDatabase()             │
│  @GET("users")                   │  ├─────────────────────────────────────┤
│  suspend fun getUsers()          │  │  @Database(                         │
│                                  │  │    entities = [User::class],        │
│  @POST("users")                  │  │    version = 1                      │
│  suspend fun createUser()        │  │  )                                  │
│                                  │  │                                     │
└──────────────────────────────────┘  │  abstract fun userDao(): UserDao    │
                   ↓                  └─────────────────────────────────────┘
                   │                                ↓
                   │                  ┌─────────────────────────────────────┐
                   │                  │      UserDao (interface)            │
                   │                  │      @Dao                           │
                   │                  ├─────────────────────────────────────┤
                   │                  │  @Query("SELECT * FROM users")      │
                   │                  │  suspend fun getAllUsers()          │
                   │                  │                                     │
                   │                  │  @Insert                            │
                   │                  │  suspend fun insertUser()           │
                   │                  └─────────────────────────────────────┘
                   ↓                                ↓
┌──────────────────────────────────┐  ┌─────────────────────────────────────┐
│       User (data class)          │  │       User (data class)             │
│       API Response Model         │  │       @Entity(tableName = "users")  │
├──────────────────────────────────┤  ├─────────────────────────────────────┤
│  data class User(                │  │  @Entity                            │
│    val id: Int,                  │  │  data class User(                   │
│    val name: String,             │  │    @PrimaryKey val id: Int,         │
│    val email: String             │  │    val name: String,                │
│  )                               │  │    val email: String                │
│                                  │  │  )                                  │
└──────────────────────────────────┘  └─────────────────────────────────────┘
                   ↓                                ↓
                   └────────────────┬───────────────┘
                                    ↓
                    ┌───────────────────────────────┐
                    │   UserViewModel (class)       │
                    │   @HiltViewModel              │
                    ├───────────────────────────────┤
                    │  @Inject constructor(         │
                    │    private val apiService,    │
                    │    private val userDao        │
                    │  )                            │
                    │                               │
                    │  fun loadFromApi()            │
                    │  fun loadFromDb()             │
                    │  fun syncData()               │
                    └───────────────────────────────┘
                                    ↓
                    ┌───────────────────────────────┐
                    │    Composable Screen          │
                    │    @Composable                │
                    ├───────────────────────────────┤
                    │  fun UserScreen(              │
                    │    viewModel: UserViewModel = │
                    │      hiltViewModel()          │
                    │  )                            │
                    └───────────────────────────────┘
```

---

## 📡 NetworkModule Code (Retrofit)

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {
    
    @Provides
    @Singleton
    fun provideRetrofit(): Retrofit {
        return Retrofit.Builder()
            .baseUrl("https://api.example.com/")
            .addConverterFactory(GsonConverterFactory.create())
            .build()
    }
    
    @Provides
    @Singleton
    fun provideApiService(retrofit: Retrofit): ApiService {
        return retrofit.create(ApiService::class.java)
    }
}
```

---

## 🗄️ DatabaseModule Code (Room)

```kotlin
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
            "my_database"
        ).build()
    }
    
    @Provides
    fun provideUserDao(database: AppDatabase): UserDao {
        return database.userDao()
    }
}
```

---

## 📝 Type Reference Table

| Component | Type | Annotation | কেন এই type? |
|-----------|------|-----------|--------------|
| **Application** | `class` | `@HiltAndroidApp` | Android entry point |
| **Hilt Module** | `object` | `@Module` `@InstallIn` | Singleton provider |
| **API Service** | `interface` | None | Retrofit implements |
| **Database** | `abstract class` | `@Database` | Room requirement |
| **DAO** | `interface` | `@Dao` | Room implements |
| **Entity** | `data class` | `@Entity` | Table structure |
| **ViewModel** | `class` | `@HiltViewModel` | UI state holder |
| **Screen** | `@Composable` | None | UI rendering |

---

## 🎯 Annotations Explained

### @Module
```
কাজ: এটা একটা Hilt Module identify করে
কোথায়: object class এর উপরে
```

### @InstallIn(SingletonComponent::class)
```
কাজ: Module কোন scope এ থাকবে define করে
SingletonComponent = পুরো app lifetime এ একটাই instance
কোথায়: @Module এর সাথে
```

### @Provides
```
কাজ: এই function dependency provide করবে
কোথায়: প্রতিটা provide function এর উপরে
```

### @Singleton
```
কাজ: শুধু একটা instance তৈরি হবে পুরো app এ
কোথায়: @Provides এর সাথে (optional কিন্তু recommended)
```

---

## ✅ Retrofit Setup Checklist

```
□ Step 1: Application class তৈরি করো
   @HiltAndroidApp
   class MyApp : Application()
   
   (AndroidManifest.xml এ android:name=".MyApp" add করো)

□ Step 2: API Interface তৈরি করো (interface)
   interface ApiService {
     @GET("users")
     suspend fun getUsers(): List<User>
   }

□ Step 3: Network Module তৈরি করো (object)
   @Module
   @InstallIn(SingletonComponent::class)
   object NetworkModule {
     @Provides @Singleton
     fun provideRetrofit(): Retrofit { ... }
     
     @Provides @Singleton
     fun provideApiService(retrofit: Retrofit): ApiService { ... }
   }

□ Step 4: ViewModel তৈরি করো (class)
   @HiltViewModel
   class UserViewModel @Inject constructor(
     private val apiService: ApiService
   ) : ViewModel()

□ Step 5: Screen এ ব্যবহার করো
   @Composable
   fun UserScreen(viewModel: UserViewModel = hiltViewModel())
```

---

## ✅ Room Setup Checklist

```
□ Step 1: Application class (Retrofit এর মতো same)

□ Step 2: Entity তৈরি করো (data class)
   @Entity(tableName = "users")
   data class User(
     @PrimaryKey val id: Int,
     val name: String,
     val email: String
   )

□ Step 3: DAO তৈরি করো (interface)
   @Dao
   interface UserDao {
     @Query("SELECT * FROM users")
     suspend fun getAllUsers(): List<User>
     
     @Insert
     suspend fun insertUser(user: User)
   }

□ Step 4: Database তৈরি করো (abstract class)
   @Database(entities = [User::class], version = 1)
   abstract class AppDatabase : RoomDatabase() {
     abstract fun userDao(): UserDao
   }

□ Step 5: Database Module তৈরি করো (object)
   @Module
   @InstallIn(SingletonComponent::class)
   object DatabaseModule {
     @Provides @Singleton
     fun provideDatabase(@ApplicationContext context): AppDatabase { ... }
     
     @Provides
     fun provideUserDao(database: AppDatabase): UserDao { ... }
   }

□ Step 6: ViewModel তৈরি করো (class)
   @HiltViewModel
   class UserViewModel @Inject constructor(
     private val userDao: UserDao
   ) : ViewModel()

□ Step 7: Screen এ ব্যবহার করো
   @Composable
   fun UserScreen(viewModel: UserViewModel = hiltViewModel())
```

---

## 🧠 Memory Tricks

### "IAO" Rule
```
I = Interface     → API Service, DAO
A = Abstract      → Database only
O = Object        → Hilt Modules
```

### "@MIP" for Modules
```
@Module           → Declares a module
@InstallIn        → Defines scope
@Provides         → Provides dependencies
```

### Singleton Pattern
```
@Singleton শুধু তখনই লাগে যখন:
✅ API Service - একটাই instance চাই
✅ Database - একটাই instance চাই
❌ DAO - Database থেকে আসে, আলাদা @Singleton লাগে না
```

---

## 📦 Dependencies (build.gradle.kts)

### Project Level
```kotlin
plugins {
    id("com.google.dagger.hilt.android") version "2.50" apply false
    id("com.google.devtools.ksp") version "1.9.22-1.0.17" apply false
}
```

### Module Level
```kotlin
plugins {
    id("com.google.dagger.hilt.android")
    id("com.google.devtools.ksp")
}

dependencies {
    // Hilt
    implementation("com.google.dagger:hilt-android:2.50")
    ksp("com.google.dagger:hilt-android-compiler:2.50")
    implementation("androidx.hilt:hilt-navigation-compose:1.1.0")
    
    // Retrofit
    implementation("com.squareup.retrofit2:retrofit:2.9.0")
    implementation("com.squareup.retrofit2:converter-gson:2.9.0")
    
    // Room
    implementation("androidx.room:room-runtime:2.6.1")
    implementation("androidx.room:room-ktx:2.6.1")
    ksp("androidx.room:room-compiler:2.6.1")
}
```

---

## 🔍 Component Scopes Reference

| Component | Lifecycle | Use Case |
|-----------|-----------|----------|
| `SingletonComponent` | App lifetime | API, Database (সবচেয়ে common) |
| `ActivityComponent` | Activity lifetime | Activity-specific dependencies |
| `ViewModelComponent` | ViewModel lifetime | ViewModel-specific dependencies |
| `ActivityRetainedComponent` | Config changes survive | Retained dependencies |

---

## 💡 Complete Code Examples

### 1. Application Class
```kotlin
@HiltAndroidApp
class MyApp : Application()
```

### 2. API Interface
```kotlin
interface ApiService {
    @GET("users")
    suspend fun getUsers(): List<User>
    
    @POST("users")
    suspend fun createUser(@Body user: User): User
    
    @GET("users/{id}")
    suspend fun getUserById(@Path("id") id: Int): User
}
```

### 3. Entity
```kotlin
@Entity(tableName = "users")
data class User(
    @PrimaryKey val id: Int,
    val name: String,
    val email: String
)
```

### 4. DAO
```kotlin
@Dao
interface UserDao {
    @Query("SELECT * FROM users")
    suspend fun getAllUsers(): List<User>
    
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertUser(user: User)
    
    @Delete
    suspend fun deleteUser(user: User)
    
    @Query("DELETE FROM users")
    suspend fun deleteAllUsers()
}
```

### 5. Database
```kotlin
@Database(
    entities = [User::class],
    version = 1,
    exportSchema = false
)
abstract class AppDatabase : RoomDatabase() {
    abstract fun userDao(): UserDao
}
```

### 6. ViewModel
```kotlin
@HiltViewModel
class UserViewModel @Inject constructor(
    private val apiService: ApiService,
    private val userDao: UserDao
) : ViewModel() {
    
    private val _users = MutableStateFlow<List<User>>(emptyList())
    val users: StateFlow<List<User>> = _users.asStateFlow()
    
    private val _isLoading = MutableStateFlow(false)
    val isLoading: StateFlow<Boolean> = _isLoading.asStateFlow()
    
    fun loadUsersFromApi() {
        viewModelScope.launch {
            _isLoading.value = true
            try {
                val userList = apiService.getUsers()
                _users.value = userList
            } catch (e: Exception) {
                // Handle error
            } finally {
                _isLoading.value = false
            }
        }
    }
    
    fun loadUsersFromDb() {
        viewModelScope.launch {
            val userList = userDao.getAllUsers()
            _users.value = userList
        }
    }
    
    fun syncUsers() {
        viewModelScope.launch {
            try {
                val apiUsers = apiService.getUsers()
                apiUsers.forEach { userDao.insertUser(it) }
                _users.value = apiUsers
            } catch (e: Exception) {
                // Handle error
            }
        }
    }
}
```

### 7. Composable Screen
```kotlin
@Composable
fun UserListScreen(
    viewModel: UserViewModel = hiltViewModel()
) {
    val users by viewModel.users.collectAsState()
    val isLoading by viewModel.isLoading.collectAsState()
    
    LaunchedEffect(Unit) {
        viewModel.loadUsersFromApi()
    }
    
    if (isLoading) {
        Box(
            modifier = Modifier.fillMaxSize(),
            contentAlignment = Alignment.Center
        ) {
            CircularProgressIndicator()
        }
    } else {
        LazyColumn(
            modifier = Modifier.fillMaxSize(),
            contentPadding = PaddingValues(16.dp)
        ) {
            items(users) { user ->
                Card(
                    modifier = Modifier
                        .fillMaxWidth()
                        .padding(vertical = 8.dp)
                ) {
                    Column(
                        modifier = Modifier.padding(16.dp)
                    ) {
                        Text(
                            text = user.name,
                            style = MaterialTheme.typography.titleMedium
                        )
                        Text(
                            text = user.email,
                            style = MaterialTheme.typography.bodyMedium
                        )
                    }
                }
            }
        }
    }
}
```

---

## ⚠️ Common Mistakes & Solutions

| ❌ Wrong | ✅ Correct | কেন? |
|---------|-----------|------|
| `class NetworkModule` | `object NetworkModule` | Singleton হতে হবে |
| `interface AppDatabase` | `abstract class AppDatabase` | Room requirement |
| `class ApiService` | `interface ApiService` | Retrofit implement করে |
| `abstract class UserDao` | `interface UserDao` | Room implement করে |
| `object UserViewModel` | `class UserViewModel` | Instance per screen লাগে |
| Missing `@Singleton` | Add `@Singleton` | Multiple instances হবে |

---

## 🐛 Troubleshooting Guide

| Error | Solution |
|-------|----------|
| Hilt component not found | Application class এ `@HiltAndroidApp` add করো |
| Cannot inject ViewModel | `@HiltViewModel` annotation check করো |
| Database instance null | Module এ `@Provides` করেছো কিনা check করো |
| API call fails | Module এ ApiService provide করেছো কিনা check করো |
| Compilation error | KSP plugin add করেছো কিনা check করো |
| Unresolved reference | Dependencies sync করো |

---

## 📚 Quick Decision Tree

```
প্রশ্ন: আমি কি তৈরি করছি?

├─ Dependency provide করছি?
│  └─ ✅ object with @Module @InstallIn ব্যবহার করো
│
├─ API endpoint define করছি?
│  └─ ✅ interface ব্যবহার করো (ApiService)
│
├─ Database table তৈরি করছি?
│  └─ ✅ data class with @Entity ব্যবহার করো
│
├─ Database operations define করছি?
│  └─ ✅ interface with @Dao ব্যবহার করো
│
├─ Room Database instance তৈরি করছি?
│  └─ ✅ abstract class with @Database ব্যবহার করো
│
└─ UI state manage করছি?
   └─ ✅ class with @HiltViewModel ব্যবহার করো
```

---

**Created by: Hasibuzzaman Chowdhury**  
**Date: December 29, 2025**  
**Version: 1.0**

---
