# Kotlin `const` Keyword - সংক্ষিপ্ত Tutorial

## `const` কি?
```kotlin
const = Compile-time constant
      = যার value compile time এই fixed থাকে

const val API_KEY = "abc123"  // ✅ Compile time এ fix
val userId = getUserId()      // ❌ Runtime এ calculate হয়
```

---

## `const` vs `val`
```kotlin
┌──────────────────────────────────────────────┐
│           const vs val                       │
└──────────────────────────────────────────────┘

const val:
✅ Compile-time constant
✅ Must be primitive or String
✅ Must be top-level or in object/companion object
✅ Value known at compile time
✅ No getter generated
✅ Faster access

val:
✅ Runtime constant
✅ Any type allowed
✅ Can be anywhere
✅ Value can be computed
✅ Getter generated
✅ Slightly slower
```

### Examples:
```kotlin
// ✅ const - Compile time constant
const val BASE_URL = "https://api.example.com"
const val API_VERSION = 1
const val TIMEOUT = 30L
const val MAX_RETRY = 3

// ✅ val - Runtime constant
val userId = getUserId()  // Computed at runtime
val timestamp = System.currentTimeMillis()
val device = Build.MODEL
```

---

## Rules - কখন `const` use করতে পারবেন
```kotlin
// ✅ Allowed types:
const val STRING = "text"           // String
const val INT = 42                  // Int
const val LONG = 100L               // Long
const val FLOAT = 3.14f             // Float
const val DOUBLE = 3.14             // Double
const val BOOLEAN = true            // Boolean

// ❌ Not allowed:
const val LIST = listOf(1, 2, 3)   // Error! ❌
const val USER = User()             // Error! ❌
const val DATE = Date()             // Error! ❌
```

### Location Rules:
```kotlin
// ✅ Top-level
const val APP_NAME = "MyApp"

// ✅ Inside object
object AppConfig {
    const val BASE_URL = "https://api.example.com"
}

// ✅ Inside companion object
class ApiService {
    companion object {
        const val TIMEOUT = 30L
    }
}

// ❌ Inside regular class (NOT allowed)
class MyClass {
    const val VALUE = 10  // Error! ❌
}

// ❌ Inside function (NOT allowed)
fun myFunction() {
    const val VALUE = 10  // Error! ❌
}
```

---

## Advantages - সুবিধা

### 1️⃣ Performance (Fast Access)
```kotlin
// const - Direct value substitution at compile time
const val MAX_SIZE = 100

fun validate(size: Int): Boolean {
    return size <= MAX_SIZE  // Replaced with: size <= 100
}

// Compiled code equivalent:
fun validate(size: Int): Boolean {
    return size <= 100  // Direct comparison! ⚡
}

────────────────────────────────────────

// val - Getter call at runtime
object Config {
    val MAX_SIZE = 100  // Has getter
}

fun validate(size: Int): Boolean {
    return size <= Config.MAX_SIZE  // Getter call
}

// const is faster! ✅
```

### 2️⃣ No Memory Allocation
```kotlin
// const - No object, just value
const val PI = 3.14159

// val - Creates object with getter
val PI = 3.14159

// const uses less memory! 💾
```

### 3️⃣ Better for Java Interop
```kotlin
// Kotlin code:
object Constants {
    const val API_KEY = "abc123"
    val USER_ID = "user_id"
}

// Java code:
String key1 = Constants.API_KEY;        // ✅ Direct access
String key2 = Constants.INSTANCE.getUSER_ID();  // ❌ Needs getter

// const is cleaner from Java! ✅
```

### 4️⃣ Annotation Parameters
```kotlin
// ✅ const can be used in annotations
const val CHANNEL_ID = "notifications"

@RequiresApi(Build.VERSION_CODES.O)
fun createNotificationChannel(context: Context) {
    val channel = NotificationChannel(
        CHANNEL_ID,  // ✅ Works!
        "Notifications",
        NotificationManager.IMPORTANCE_DEFAULT
    )
}

────────────────────────────────────────

// ❌ val cannot be used in annotations
val CHANNEL_ID = "notifications"

fun createNotificationChannel(context: Context) {
    val channel = NotificationChannel(
        CHANNEL_ID,  // ❌ Error in some contexts!
        "Notifications",
        NotificationManager.IMPORTANCE_DEFAULT
    )
}
```

---

## Real World Examples

### 1️⃣ API Configuration
```kotlin
object ApiConfig {
    const val BASE_URL = "https://api.example.com"
    const val API_VERSION = "v1"
    const val TIMEOUT = 30L
    const val MAX_RETRY = 3
    
    // Full URL
    const val USERS_ENDPOINT = "$BASE_URL/$API_VERSION/users"
}

// Use:
Retrofit.Builder()
    .baseUrl(ApiConfig.BASE_URL)
    .connectTimeout(ApiConfig.TIMEOUT, TimeUnit.SECONDS)
    .build()
```

### 2️⃣ SharedPreferences Keys
```kotlin
object PrefsKeys {
    const val KEY_USER_TOKEN = "user_token"
    const val KEY_USER_ID = "user_id"
    const val KEY_IS_LOGGED_IN = "is_logged_in"
    const val KEY_THEME = "theme"
}

// Use:
prefs.edit()
    .putString(PrefsKeys.KEY_USER_TOKEN, token)
    .putString(PrefsKeys.KEY_USER_ID, userId)
    .apply()
```

### 3️⃣ Intent/Bundle Keys
```kotlin
class UserDetailActivity : AppCompatActivity() {
    
    companion object {
        private const val EXTRA_USER_ID = "extra_user_id"
        private const val EXTRA_USER_NAME = "extra_user_name"
        
        fun newIntent(context: Context, userId: String, userName: String): Intent {
            return Intent(context, UserDetailActivity::class.java).apply {
                putExtra(EXTRA_USER_ID, userId)
                putExtra(EXTRA_USER_NAME, userName)
            }
        }
    }
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        val userId = intent.getStringExtra(EXTRA_USER_ID)
        val userName = intent.getStringExtra(EXTRA_USER_NAME)
    }
}
```

### 4️⃣ Database Constants
```kotlin
object DatabaseConstants {
    const val DATABASE_NAME = "app_database"
    const val DATABASE_VERSION = 1
    
    // Table names
    const val TABLE_USERS = "users"
    const val TABLE_POSTS = "posts"
    
    // Column names
    const val COLUMN_ID = "id"
    const val COLUMN_NAME = "name"
    const val COLUMN_EMAIL = "email"
}

// Use in Room:
@Database(
    entities = [User::class, Post::class],
    version = DatabaseConstants.DATABASE_VERSION
)
abstract class AppDatabase : RoomDatabase()
```

### 5️⃣ Validation Constants
```kotlin
object ValidationRules {
    const val MIN_PASSWORD_LENGTH = 8
    const val MAX_PASSWORD_LENGTH = 20
    const val MIN_USERNAME_LENGTH = 3
    const val MAX_USERNAME_LENGTH = 15
    const val MIN_AGE = 18
    const val MAX_AGE = 100
}

// Use:
fun validatePassword(password: String): Boolean {
    return password.length in 
        ValidationRules.MIN_PASSWORD_LENGTH..ValidationRules.MAX_PASSWORD_LENGTH
}

fun validateUsername(username: String): Boolean {
    return username.length >= ValidationRules.MIN_USERNAME_LENGTH
}
```

### 6️⃣ Network Response Codes
```kotlin
object HttpStatus {
    const val OK = 200
    const val CREATED = 201
    const val NO_CONTENT = 204
    const val BAD_REQUEST = 400
    const val UNAUTHORIZED = 401
    const val FORBIDDEN = 403
    const val NOT_FOUND = 404
    const val SERVER_ERROR = 500
}

// Use:
when (response.code) {
    HttpStatus.OK -> // Success
    HttpStatus.UNAUTHORIZED -> // Login required
    HttpStatus.NOT_FOUND -> // Not found
    HttpStatus.SERVER_ERROR -> // Server error
}
```

---

## When to Use `const` vs `val`
```kotlin
// ✅ Use const for:
const val BASE_URL = "https://api.example.com"    // Fixed URL
const val TIMEOUT = 30L                            // Fixed number
const val DEBUG = true                             // Fixed boolean
const val APP_NAME = "MyApp"                       // Fixed string

// ✅ Use val for:
val userId = getUserId()                           // Computed
val timestamp = System.currentTimeMillis()         // Runtime value
val deviceModel = Build.MODEL                      // System value
val userList = listOf<User>()                      // Complex type
```

---

## Common Mistakes
```kotlin
// ❌ Error: const must be primitive or String
const val USERS = listOf("user1", "user2")  // Error!

// ✅ Fix: Use val
val USERS = listOf("user1", "user2")

────────────────────────────────────────

// ❌ Error: const must be in object/companion object/top-level
class MyClass {
    const val VALUE = 10  // Error!
}

// ✅ Fix: Move to companion object
class MyClass {
    companion object {
        const val VALUE = 10  // Works!
    }
}

────────────────────────────────────────

// ❌ Error: const must have known value at compile time
const val TIMESTAMP = System.currentTimeMillis()  // Error!

// ✅ Fix: Use val
val TIMESTAMP = System.currentTimeMillis()
```

---

## Summary
```kotlin
╔══════════════════════════════════════════╗
║        const Keyword - Key Points        ║
╚══════════════════════════════════════════╝

What:
  Compile-time constant (value fixed at compile time)

Allowed Types:
  ✅ String, Int, Long, Float, Double, Boolean
  ❌ List, Array, Object, etc.

Location:
  ✅ Top-level
  ✅ Inside object
  ✅ Inside companion object
  ❌ Inside regular class
  ❌ Inside function

Advantages:
  ⚡ Faster (direct value substitution)
  💾 Less memory (no getter)
  🔗 Better Java interop
  📝 Can use in annotations

Use Cases:
  • API URLs, keys
  • SharedPreferences keys
  • Database constants
  • Validation rules
  • HTTP status codes

Remember:
  const = Compile-time known ⚡
  val = Runtime value 🏃
```

---

## Quick Reference
```kotlin
// Top-level const
const val APP_NAME = "MyApp"

// In object
object Config {
    const val BASE_URL = "https://api.example.com"
}

// In companion object
class User {
    companion object {
        const val DEFAULT_ROLE = "user"
    }
}

// Use:
println(APP_NAME)
println(Config.BASE_URL)
println(User.DEFAULT_ROLE)
```

---

*Happy Coding! 🚀*

*Last Updated: December 2024*
