# Kotlin: object, companion object & Singleton

## 📚 সূচিপত্র
1. [object Keyword](#object-keyword)
2. [companion object](#companion-object)
3. [Singleton Pattern](#singleton-pattern)
4. [Real World Examples](#real-world-examples)

---

## object Keyword

### `object` কি?
```kotlin
object = Built-in Singleton

একবার তৈরি হয় + সবসময় একই instance
```

### Basic Example:
```kotlin
// Regular class (multiple instances)
class Calculator {
    fun add(a: Int, b: Int) = a + b
}

val calc1 = Calculator()  // New instance
val calc2 = Calculator()  // Another new instance
println(calc1 === calc2)  // false (different objects)

────────────────────────────────────────

// object (single instance)
object MathUtils {
    fun add(a: Int, b: Int) = a + b
}

// Direct access (no instance creation needed)
MathUtils.add(5, 3)  // 8

// Always same instance
val ref1 = MathUtils
val ref2 = MathUtils
println(ref1 === ref2)  // true (same object)
```

### কখন Use করবেন?

#### 1️⃣ Constants/Configuration
```kotlin
object AppConfig {
    const val BASE_URL = "https://api.example.com"
    const val API_KEY = "your_api_key"
    const val TIMEOUT = 30L
    const val DATABASE_NAME = "app_database"
}

// Use:
Retrofit.Builder()
    .baseUrl(AppConfig.BASE_URL)
    .build()
```

#### 2️⃣ Utility Functions
```kotlin
object DateUtils {
    fun getCurrentDate(): String {
        return SimpleDateFormat("yyyy-MM-dd", Locale.getDefault())
            .format(Date())
    }
    
    fun formatDate(timestamp: Long): String {
        return SimpleDateFormat("dd MMM yyyy", Locale.getDefault())
            .format(Date(timestamp))
    }
}

// Use:
val today = DateUtils.getCurrentDate()
val formatted = DateUtils.formatDate(System.currentTimeMillis())
```

#### 3️⃣ Logger/Analytics
```kotlin
object AppLogger {
    private const val TAG = "AppLogger"
    
    fun d(message: String) {
        if (BuildConfig.DEBUG) {
            Log.d(TAG, message)
        }
    }
    
    fun e(message: String, throwable: Throwable? = null) {
        Log.e(TAG, message, throwable)
    }
}

// Use anywhere:
AppLogger.d("User logged in")
AppLogger.e("API failed", exception)
```

---

## companion object

### `companion object` কি?
```kotlin
companion object = Class এর ভিতরে static-like members

Java এর static এর মতো কাজ করে
```

### Basic Example:
```kotlin
class User(val name: String, val email: String) {
    
    // companion object - class level members
    companion object {
        const val DEFAULT_COUNTRY = "Bangladesh"
        
        fun create(name: String, email: String): User {
            return User(name, email)
        }
        
        fun createGuest(): User {
            return User("Guest", "guest@example.com")
        }
    }
}

// Use without creating instance:
val defaultCountry = User.DEFAULT_COUNTRY
val user = User.create("Hasib", "hasib@example.com")
val guest = User.createGuest()
```

### কখন Use করবেন?

#### 1️⃣ Factory Methods
```kotlin
data class User(
    val id: String,
    val name: String,
    val email: String
) {
    companion object {
        // Create from API response
        fun fromJson(json: JSONObject): User {
            return User(
                id = json.getString("id"),
                name = json.getString("name"),
                email = json.getString("email")
            )
        }
        
        // Create from database
        fun fromEntity(entity: UserEntity): User {
            return User(
                id = entity.id,
                name = entity.name,
                email = entity.email
            )
        }
        
        // Create default/empty
        fun empty(): User {
            return User("", "", "")
        }
    }
}

// Use:
val user1 = User.fromJson(jsonObject)
val user2 = User.fromEntity(entity)
val emptyUser = User.empty()
```

#### 2️⃣ Constants for Class
```kotlin
data class ApiResponse<T>(
    val status: String,
    val data: T?,
    val message: String?
) {
    companion object {
        const val STATUS_SUCCESS = "success"
        const val STATUS_ERROR = "error"
        const val STATUS_LOADING = "loading"
    }
}

// Use:
when (response.status) {
    ApiResponse.STATUS_SUCCESS -> // handle success
    ApiResponse.STATUS_ERROR -> // handle error
}
```

#### 3️⃣ Intent Keys (Android)
```kotlin
class UserDetailActivity : AppCompatActivity() {
    
    companion object {
        private const val EXTRA_USER_ID = "user_id"
        private const val EXTRA_USER_NAME = "user_name"
        
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

// Use:
val intent = UserDetailActivity.newIntent(context, "123", "Hasib")
startActivity(intent)
```

#### 4️⃣ Fragment Arguments
```kotlin
class UserFragment : Fragment() {
    
    companion object {
        private const val ARG_USER_ID = "user_id"
        
        fun newInstance(userId: String): UserFragment {
            return UserFragment().apply {
                arguments = Bundle().apply {
                    putString(ARG_USER_ID, userId)
                }
            }
        }
    }
    
    private val userId: String
        get() = arguments?.getString(ARG_USER_ID) ?: ""
}

// Use:
val fragment = UserFragment.newInstance("123")
```

---

## Singleton Pattern

### Manual Singleton (Old Way)
```kotlin
class DatabaseManager private constructor() {
    
    companion object {
        @Volatile
        private var instance: DatabaseManager? = null
        
        fun getInstance(): DatabaseManager {
            return instance ?: synchronized(this) {
                instance ?: DatabaseManager().also { instance = it }
            }
        }
    }
    
    fun query() { /* ... */ }
}

// Use:
val db = DatabaseManager.getInstance()
```

### Using `object` (Kotlin Way - Better!)
```kotlin
object DatabaseManager {
    fun query() { /* ... */ }
}

// Use directly:
DatabaseManager.query()

// Much simpler! ✅
```

### Lazy Initialization Singleton
```kotlin
class ImageLoader private constructor(private val context: Context) {
    
    companion object {
        @Volatile
        private var instance: ImageLoader? = null
        
        fun getInstance(context: Context): ImageLoader {
            return instance ?: synchronized(this) {
                instance ?: ImageLoader(context.applicationContext).also {
                    instance = it
                }
            }
        }
    }
    
    fun loadImage(url: String, imageView: ImageView) {
        // Load image using Glide/Coil
    }
}

// Use:
ImageLoader.getInstance(context).loadImage(url, imageView)
```

---

## Real World Examples

### 1️⃣ Preferences Manager
```kotlin
object PreferencesManager {
    private const val PREFS_NAME = "app_preferences"
    private const val KEY_USER_TOKEN = "user_token"
    private const val KEY_USER_ID = "user_id"
    
    private lateinit var prefs: SharedPreferences
    
    fun init(context: Context) {
        prefs = context.getSharedPreferences(PREFS_NAME, Context.MODE_PRIVATE)
    }
    
    fun saveToken(token: String) {
        prefs.edit().putString(KEY_USER_TOKEN, token).apply()
    }
    
    fun getToken(): String? {
        return prefs.getString(KEY_USER_TOKEN, null)
    }
    
    fun saveUserId(userId: String) {
        prefs.edit().putString(KEY_USER_ID, userId).apply()
    }
    
    fun getUserId(): String? {
        return prefs.getString(KEY_USER_ID, null)
    }
    
    fun clear() {
        prefs.edit().clear().apply()
    }
}

// In Application class:
class MyApp : Application() {
    override fun onCreate() {
        super.onCreate()
        PreferencesManager.init(this)
    }
}

// Use anywhere:
PreferencesManager.saveToken("abc123")
val token = PreferencesManager.getToken()
```

### 2️⃣ Network Manager
```kotlin
object NetworkManager {
    
    fun isNetworkAvailable(context: Context): Boolean {
        val connectivityManager = context.getSystemService(
            Context.CONNECTIVITY_SERVICE
        ) as ConnectivityManager
        
        val network = connectivityManager.activeNetwork ?: return false
        val capabilities = connectivityManager.getNetworkCapabilities(network)
        
        return capabilities?.hasCapability(
            NetworkCapabilities.NET_CAPABILITY_INTERNET
        ) ?: false
    }
    
    fun isWifiConnected(context: Context): Boolean {
        val connectivityManager = context.getSystemService(
            Context.CONNECTIVITY_SERVICE
        ) as ConnectivityManager
        
        val network = connectivityManager.activeNetwork ?: return false
        val capabilities = connectivityManager.getNetworkCapabilities(network)
        
        return capabilities?.hasTransport(
            NetworkCapabilities.TRANSPORT_WIFI
        ) ?: false
    }
}

// Use:
if (NetworkManager.isNetworkAvailable(context)) {
    // Make API call
} else {
    // Show offline message
}
```

### 3️⃣ Event Bus (Simple)
```kotlin
object EventBus {
    
    private val listeners = mutableMapOf<String, MutableList<(Any) -> Unit>>()
    
    fun subscribe(event: String, listener: (Any) -> Unit) {
        listeners.getOrPut(event) { mutableListOf() }.add(listener)
    }
    
    fun post(event: String, data: Any) {
        listeners[event]?.forEach { it(data) }
    }
    
    fun unsubscribe(event: String) {
        listeners.remove(event)
    }
}

// Use:
// Subscribe
EventBus.subscribe("user_logged_in") { data ->
    val user = data as User
    println("User logged in: ${user.name}")
}

// Post
EventBus.post("user_logged_in", user)
```

### 4️⃣ Data Class with companion object
```kotlin
data class Product(
    val id: String,
    val name: String,
    val price: Double,
    val imageUrl: String
) {
    companion object {
        // Sample data for testing
        fun getSampleProducts(): List<Product> {
            return listOf(
                Product("1", "Laptop", 50000.0, "url1"),
                Product("2", "Phone", 30000.0, "url2"),
                Product("3", "Tablet", 25000.0, "url3")
            )
        }
        
        // Validation
        fun isValidPrice(price: Double): Boolean {
            return price > 0
        }
        
        // Formatting
        fun formatPrice(price: Double): String {
            return "৳${String.format("%.2f", price)}"
        }
    }
}

// Use:
val sampleProducts = Product.getSampleProducts()
val isValid = Product.isValidPrice(100.0)
val formatted = Product.formatPrice(50000.0)  // ৳50000.00
```

### 5️⃣ Repository with companion object
```kotlin
class UserRepository @Inject constructor(
    private val apiService: ApiService,
    private val userDao: UserDao
) {
    companion object {
        private const val CACHE_DURATION = 5 * 60 * 1000L  // 5 minutes
        
        // Helper function
        fun isCacheValid(timestamp: Long): Boolean {
            return System.currentTimeMillis() - timestamp < CACHE_DURATION
        }
    }
    
    suspend fun getUsers(): List<User> {
        // Use companion object function
        val cachedUsers = userDao.getAllUsers()
        if (cachedUsers.isNotEmpty() && isCacheValid(cachedUsers[0].timestamp)) {
            return cachedUsers
        }
        
        // Fetch from API
        val users = apiService.getUsers()
        userDao.insertAll(users)
        return users
    }
}
```

---

## Quick Comparison
```kotlin
┌────────────────────────────────────────────────────┐
│     object vs companion object vs Singleton        │
└────────────────────────────────────────────────────┘

object:
✅ Standalone singleton
✅ Global access
✅ No class needed
Example: AppConfig, Logger

companion object:
✅ Inside a class
✅ Access via ClassName
✅ Factory methods, constants
Example: User.create(), Fragment.newInstance()

Manual Singleton:
✅ Needs context/parameters
✅ Lazy initialization
✅ More control
Example: DatabaseManager, ImageLoader
```

---

## Best Practices

### ✅ Use `object` for:
```kotlin
// Configuration
object AppConfig { }

// Utilities
object StringUtils { }

// Logger
object AppLogger { }

// Simple managers (no context needed)
object EventBus { }
```

### ✅ Use `companion object` for:
```kotlin
// Factory methods
class User {
    companion object {
        fun create() { }
    }
}

// Constants related to class
class ApiResponse {
    companion object {
        const val STATUS_SUCCESS = "success"
    }
}

// Intent/Fragment creation
class MyActivity {
    companion object {
        fun newIntent() { }
    }
}
```

### ✅ Use Manual Singleton for:
```kotlin
// Needs context
class DatabaseManager(context: Context)

// Complex initialization
class NetworkClient(config: Config)

// Needs parameters
class CacheManager(maxSize: Long)
```

---

## Summary
```kotlin
╔═══════════════════════════════════════════════╗
║      object, companion object & Singleton     ║
╚═══════════════════════════════════════════════╝

object:
  • Built-in singleton
  • Global access
  • No instance creation
  • Use: Config, Utils, Logger

companion object:
  • Static-like members
  • Inside class
  • Factory methods
  • Use: Constants, Factories

Singleton:
  • Custom pattern
  • Needs context/params
  • More control
  • Use: Managers with dependencies

Remember:
  object = Simplest ✅
  companion object = Class-related ✅
  Manual Singleton = Complex needs ✅
```

---

## Quick Reference
```kotlin
// 1. object
object MyObject {
    fun doSomething() { }
}
MyObject.doSomething()

// 2. companion object
class MyClass {
    companion object {
        fun create() { }
    }
}
MyClass.create()

// 3. Manual singleton
class MySingleton private constructor() {
    companion object {
        private var instance: MySingleton? = null
        fun getInstance(): MySingleton {
            return instance ?: MySingleton().also { instance = it }
        }
    }
}
MySingleton.getInstance()
```

---

*Happy Coding! 🚀*

*Last Updated: December 2024*
