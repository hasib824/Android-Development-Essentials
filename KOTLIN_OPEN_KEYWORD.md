# Kotlin `open` Keyword - সহজ Tutorial

## `open` কি?
```kotlin
open = "আমাকে extend/override করতে পারো" 🔓

Without open: final (locked) 🔒
With open: extendable/overridable 🔓
```

---

## Kotlin এর Rule
```kotlin
Default in Kotlin:
class MyClass { }          // final (extend করা যাবে না)
fun myFunction() { }       // final (override করা যাবে না)

Java তে সব open থাকে, Kotlin এ manually open করতে হয়
```

---

## Basic Examples

### Class এ `open`
```kotlin
// ❌ Without open
class Animal {
    fun eat() = println("Eating")
}

class Dog : Animal()  // Error! Cannot inherit ❌

────────────────────────────────────────

// ✅ With open
open class Animal {
    fun eat() = println("Eating")
}

class Dog : Animal()  // Works! ✅
```

### Function এ `open`
```kotlin
open class Animal {
    // ❌ Cannot override
    fun makeSound() = println("Sound")
}

class Dog : Animal() {
    override fun makeSound()  // Error! ❌
}

────────────────────────────────────────

open class Animal {
    // ✅ Can override
    open fun makeSound() = println("Sound")
}

class Dog : Animal() {
    override fun makeSound() = println("Bark!")  // Works! ✅
}
```

---

## কখন Use করবেন? - Real Examples

### 1️⃣ Base ViewModel তৈরি করা
```kotlin
// Base ViewModel - সব ViewModel এর common logic
open class BaseViewModel : ViewModel() {
    
    private val _isLoading = MutableStateFlow(false)
    val isLoading = _isLoading.asStateFlow()
    
    open fun showLoading() {
        _isLoading.value = true
    }
    
    open fun hideLoading() {
        _isLoading.value = false
    }
    
    open fun handleError(error: Throwable) {
        // Default error handling
        hideLoading()
        Log.e("Error", error.message ?: "Unknown")
    }
}

// Specific ViewModels inherit করে
class UserViewModel : BaseViewModel() {
    
    fun loadUsers() {
        viewModelScope.launch {
            showLoading()  // ✅ Base থেকে use করছি
            try {
                // API call
            } catch (e: Exception) {
                handleError(e)  // ✅ Base থেকে use করছি
            }
        }
    }
    
    // Custom error handling চাইলে override করতে পারি
    override fun handleError(error: Throwable) {
        when (error) {
            is IOException -> showToast("No internet")
            else -> super.handleError(error)
        }
    }
}

class ProductViewModel : BaseViewModel() {
    fun loadProducts() {
        showLoading()  // ✅ Base থেকে reuse
        // ...
    }
}
```

### 2️⃣ Base Repository Pattern
```kotlin
// Common API call logic সব Repository তে
open class BaseRepository {
    
    // All repositories এ এই method use করতে পারবে
    open suspend fun <T> safeApiCall(
        apiCall: suspend () -> T
    ): Result<T> {
        return try {
            Result.success(apiCall())
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}

// User Repository
class UserRepository @Inject constructor(
    private val api: ApiService
) : BaseRepository() {
    
    suspend fun getUsers(): Result<List<User>> {
        return safeApiCall {  // ✅ Base method reuse
            api.getUsers()
        }
    }
    
    suspend fun createUser(user: User): Result<User> {
        return safeApiCall {  // ✅ Same method
            api.createUser(user)
        }
    }
}

// Product Repository
class ProductRepository @Inject constructor(
    private val api: ApiService
) : BaseRepository() {
    
    suspend fun getProducts(): Result<List<Product>> {
        return safeApiCall {  // ✅ Reusing same logic
            api.getProducts()
        }
    }
}
```

### 3️⃣ Custom View তৈরি
```kotlin
// Base Button - common functionality
open class BaseButton @JvmOverloads constructor(
    context: Context,
    attrs: AttributeSet? = null
) : MaterialButton(context, attrs) {
    
    open fun showLoading() {
        isEnabled = false
        text = "Loading..."
    }
    
    open fun hideLoading() {
        isEnabled = true
    }
}

// Specific button types
class PrimaryButton @JvmOverloads constructor(
    context: Context,
    attrs: AttributeSet? = null
) : BaseButton(context, attrs) {
    
    init {
        setBackgroundColor(Color.BLUE)
    }
    
    // Custom loading চাইলে override
    override fun showLoading() {
        super.showLoading()
        // Add spinner icon
    }
}

class DangerButton @JvmOverloads constructor(
    context: Context,
    attrs: AttributeSet? = null
) : BaseButton(context, attrs) {
    
    init {
        setBackgroundColor(Color.RED)
    }
}
```

### 4️⃣ Adapter Pattern
```kotlin
// Base Adapter - common ViewHolder logic
open class BaseAdapter<T> : RecyclerView.Adapter<BaseAdapter.BaseViewHolder>() {
    
    protected val items = mutableListOf<T>()
    
    open fun setItems(newItems: List<T>) {
        items.clear()
        items.addAll(newItems)
        notifyDataSetChanged()
    }
    
    open fun addItem(item: T) {
        items.add(item)
        notifyItemInserted(items.size - 1)
    }
    
    override fun getItemCount() = items.size
    
    open class BaseViewHolder(view: View) : RecyclerView.ViewHolder(view)
}

// User Adapter
class UserAdapter : BaseAdapter<User>() {
    
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): BaseViewHolder {
        // Create view holder
    }
    
    override fun onBindViewHolder(holder: BaseViewHolder, position: Int) {
        val user = items[position]
        // Bind user data
    }
    
    // Custom method
    fun updateUser(position: Int, user: User) {
        items[position] = user
        notifyItemChanged(position)
    }
}
```

### 5️⃣ Data Models (Less Common but Useful)
```kotlin
// Base User
open class User(
    open val id: String,
    open val name: String,
    open val email: String
)

// Extended User types
data class PremiumUser(
    override val id: String,
    override val name: String,
    override val email: String,
    val subscriptionEndDate: String
) : User(id, name, email)

data class AdminUser(
    override val id: String,
    override val name: String,
    override val email: String,
    val permissions: List<String>
) : User(id, name, email)

// Polymorphic handling
fun displayUser(user: User) {
    when (user) {
        is PremiumUser -> println("⭐ ${user.name}")
        is AdminUser -> println("👑 ${user.name}")
        else -> println(user.name)
    }
}
```

---

## `open` vs `abstract`
```kotlin
┌──────────────────────────────────────────────┐
│         open vs abstract                     │
└──────────────────────────────────────────────┘

open class Animal {
    open fun sound() { }     // Implementation আছে
}
✅ Instance: Animal() করা যায়
✅ Override: Optional

abstract class Animal {
    abstract fun sound()     // Implementation নেই
}
❌ Instance: Animal() করা যাবে না
✅ Override: Mandatory
```

### Example:
```kotlin
// open - Optional override
open class Repository {
    open fun fetchData() {
        // Default implementation
    }
}

class UserRepository : Repository() {
    // Override না করলেও চলবে
}

────────────────────────────────────────

// abstract - Mandatory override
abstract class Repository {
    abstract fun fetchData()  // Must implement
}

class UserRepository : Repository() {
    override fun fetchData() {  // Must override!
        // Implementation
    }
}
```

---

## Common Pattern - Android

### Complete Example: Repository + ViewModel
```kotlin
// 1. Base Repository
open class BaseRepository {
    open suspend fun <T> executeApiCall(
        call: suspend () -> T
    ): Result<T> {
        return try {
            Result.success(call())
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}

// 2. Specific Repository
class UserRepository @Inject constructor(
    private val api: ApiService
) : BaseRepository() {
    
    suspend fun getUsers() = executeApiCall {
        api.getUsers()
    }
}

// 3. Base ViewModel
open class BaseViewModel : ViewModel() {
    
    protected val _isLoading = MutableStateFlow(false)
    val isLoading = _isLoading.asStateFlow()
    
    open fun showLoading() {
        _isLoading.value = true
    }
    
    open fun hideLoading() {
        _isLoading.value = false
    }
}

// 4. Specific ViewModel
@HiltViewModel
class UserViewModel @Inject constructor(
    private val repository: UserRepository
) : BaseViewModel() {
    
    private val _users = MutableStateFlow<List<User>>(emptyList())
    val users = _users.asStateFlow()
    
    fun loadUsers() {
        viewModelScope.launch {
            showLoading()  // ✅ Base method
            repository.getUsers().fold(
                onSuccess = { _users.value = it },
                onFailure = { /* handle */ }
            )
            hideLoading()  // ✅ Base method
        }
    }
}
```

---

## কখন Use করবেন না?
```kotlin
// ❌ Don't use open for utility classes
class MathUtils {  // Keep final
    fun add(a: Int, b: Int) = a + b
}

// ❌ Don't use open for data classes (usually)
data class User(val name: String)  // Keep final

// ❌ Don't use open for singletons
object AppConfig {  // Can't be open anyway
    const val BASE_URL = "..."
}

// ✅ Use sealed for fixed hierarchies
sealed class Result {
    data class Success(val data: String) : Result()
    data class Error(val error: String) : Result()
}
```

---

## Quick Reference
```kotlin
// Basic syntax
open class MyClass                    // Class
open fun myFunction()                 // Function
open val myProperty: String          // Property

// Inheritance
open class Animal
class Dog : Animal()                 // Inherit

// Override
open class Animal {
    open fun sound() { }
}
class Dog : Animal() {
    override fun sound() { }         // Override
}

// Final override (stop further overriding)
open class Dog : Animal() {
    final override fun sound() { }   // No more override
}
```

---

## Summary
```
╔══════════════════════════════════════════╗
║      open Keyword - Key Points          ║
╚══════════════════════════════════════════╝

What:
  Permission to extend/override

When to Use:
  ✅ Base classes (ViewModel, Repository)
  ✅ Reusable components
  ✅ Template patterns
  ✅ Framework/Library

Common Use Cases:
  • BaseViewModel
  • BaseRepository
  • Custom Views
  • Base Adapters
  
Don't Use:
  ❌ Utility classes
  ❌ Data classes
  ❌ Security-critical code

Remember:
  Kotlin = Closed by default 🔒
  Open only when needed 🔓
```

---

*Happy Coding! 🚀*

*Last Updated: December 2024*
