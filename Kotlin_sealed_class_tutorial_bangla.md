# Kotlin Sealed Class - সম্পূর্ণ বাংলা টিউটোরিয়াল

## সূচিপত্র
1. [Sealed Class কী?](#sealed-class-কী)
2. [কেন ব্যবহার করবেন?](#কেন-ব্যবহার-করবেন)
3. [Sealed Class এর সুবিধা](#sealed-class-এর-সুবিধা)
4. [Sealed Class vs Sealed Interface](#sealed-class-vs-sealed-interface)
5. [বাস্তব উদাহরণ](#বাস্তব-উদাহরণ)
6. [Best Practices](#best-practices)

---

## Sealed Class কী?

Sealed Class হলো একটি **বিশেষ ধরনের class** যার **সীমিত সংখ্যক subclass** থাকতে পারে। এটি একটি "**সীলমোহর করা**" class যার সব সম্ভাব্য উত্তরাধিকারী (inheritor) আগে থেকেই জানা।

### সহজ ভাষায়:
ধরুন, আপনার কাছে একটি বাক্স আছে। Sealed Class মানে হলো সেই বাক্সে **কী কী জিনিস থাকতে পারে তা আগে থেকেই নির্ধারিত**। বাক্সের বাইরে থেকে নতুন কিছু ঢোকানো যাবে না।

### সিনট্যাক্স:
```kotlin
sealed class Result {
    data class Success(val data: String) : Result()
    data class Error(val message: String) : Result()
    data object Loading : Result()
}
```

---

## কেন ব্যবহার করবেন?

### ১. **Type-Safe When Expression**
Sealed class ব্যবহার করলে `when` expression এ সব case কভার করা **বাধ্যতামূলক**। `else` ব্রাঞ্চের দরকার নেই।

```kotlin
fun handleResult(result: Result) {
    when (result) {
        is Result.Success -> println("সফল: ${result.data}")
        is Result.Error -> println("ভুল: ${result.message}")
        Result.Loading -> println("লোডিং...")
    }
    // else ব্রাঞ্চ লাগবে না! কম্পাইলার নিশ্চিত করে সব case কভার হয়েছে
}
```

### ২. **Restricted Hierarchies**
নির্দিষ্ট সংখ্যক state/type represent করতে চাইলে sealed class পারফেক্ট।

### ৩. **Compile-Time Safety**
নতুন subclass add করলে কম্পাইলার সব `when` expression এ error দেখাবে যেখানে নতুন case handle করতে হবে।

---

## Sealed Class এর সুবিধা

### ✅ **১. Exhaustive When Expression**
```kotlin
sealed class NetworkResponse {
    data class Success(val data: String) : NetworkResponse()
    data class Failure(val error: Throwable) : NetworkResponse()
    data object NetworkError : NetworkResponse()
}

fun processResponse(response: NetworkResponse): String {
    return when (response) {
        is NetworkResponse.Success -> "ডেটা পেয়েছি: ${response.data}"
        is NetworkResponse.Failure -> "এরর: ${response.error.message}"
        NetworkResponse.NetworkError -> "ইন্টারনেট নেই"
    }
    // else দরকার নেই - কম্পাইলার check করে!
}
```

### ✅ **২. পরিষ্কার State Management**
```kotlin
sealed class UiState {
    data object Idle : UiState()
    data object Loading : UiState()
    data class Success(val items: List<String>) : UiState()
    data class Error(val message: String) : UiState()
}

// ViewModel এ ব্যবহার
class MyViewModel : ViewModel() {
    private val _uiState = MutableStateFlow<UiState>(UiState.Idle)
    val uiState: StateFlow<UiState> = _uiState
    
    fun loadData() {
        viewModelScope.launch {
            _uiState.value = UiState.Loading
            try {
                val data = repository.getData()
                _uiState.value = UiState.Success(data)
            } catch (e: Exception) {
                _uiState.value = UiState.Error(e.message ?: "Unknown error")
            }
        }
    }
}
```

### ✅ **৩. Better than Enum**
Enum এর মতো কিন্তু প্রতিটি subclass **ভিন্ন ভিন্ন data hold** করতে পারে।

```kotlin
// ❌ Enum দিয়ে এটা সম্ভব না
enum class Status {
    SUCCESS, // data রাখতে পারবো না
    ERROR    // error message রাখতে পারবো না
}

// ✅ Sealed Class দিয়ে সম্ভব
sealed class Status {
    data class Success(val data: String) : Status()
    data class Error(val message: String, val code: Int) : Status()
}
```

---

## Sealed Class vs Sealed Interface

### 📦 **Sealed Class**

**কখন ব্যবহার করবেন:**
- যখন **common behavior/properties** share করতে হবে
- যখন **single inheritance** যথেষ্ট
- যখন **state** manage করতে হবে

```kotlin
sealed class PaymentMethod(val fee: Double) {
    data class CreditCard(val cardNumber: String) : PaymentMethod(2.5)
    data class BKash(val phoneNumber: String) : PaymentMethod(1.5)
    data class Nagad(val phoneNumber: String) : PaymentMethod(1.0)
    
    // Common function সব subclass এ পাবে
    fun calculateFee(amount: Double): Double {
        return amount * (fee / 100)
    }
}

// ব্যবহার
val payment = PaymentMethod.BKash("01711223344")
val totalFee = payment.calculateFee(1000.0) // 15 টাকা ফি
```

### 🔌 **Sealed Interface**

**কখন ব্যবহার করবেন:**
- যখন **multiple inheritance** লাগবে
- যখন শুধু **contract define** করতে হবে, implementation নয়
- যখন আরো **flexible** architecture চাই

```kotlin
sealed interface UserAction
sealed interface AdminAction

// একসাথে multiple sealed interface implement করা যায়
data class DeleteUser(val userId: String) : UserAction, AdminAction
data class UpdateProfile(val name: String) : UserAction
data class GenerateReport(val type: String) : AdminAction

fun handleAction(action: UserAction) {
    when (action) {
        is DeleteUser -> println("ইউজার ডিলিট: ${action.userId}")
        is UpdateProfile -> println("প্রোফাইল আপডেট: ${action.name}")
    }
}
```

### 🆚 **তুলনা**

| Feature | Sealed Class | Sealed Interface |
|---------|-------------|------------------|
| Common properties | ✅ হ্যাঁ | ❌ না |
| Common functions | ✅ হ্যাঁ (implementation সহ) | ⚠️ হ্যাঁ (শুধু declaration) |
| Multiple inheritance | ❌ না | ✅ হ্যাঁ |
| Constructor | ✅ থাকতে পারে | ❌ থাকে না |
| State hold করা | ✅ সহজ | ⚠️ জটিল |

---

## বাস্তব উদাহরণ

### 🎯 **উদাহরণ ১: API Response Handling**

```kotlin
sealed class ApiResponse<out T> {
    data class Success<T>(val data: T) : ApiResponse<T>()
    data class Error(
        val message: String,
        val code: Int,
        val exception: Throwable? = null
    ) : ApiResponse<Nothing>()
    data object Loading : ApiResponse<Nothing>()
    
    // Sealed class এর ভিতরে function
    fun <R> map(transform: (T) -> R): ApiResponse<R> {
        return when (this) {
            is Success -> Success(transform(data))
            is Error -> this
            Loading -> this
        }
    }
    
    fun isSuccess(): Boolean = this is Success
    fun isError(): Boolean = this is Error
    fun isLoading(): Boolean = this is Loading
    
    fun getDataOrNull(): T? = (this as? Success)?.data
}

// ব্যবহার
class UserRepository {
    suspend fun getUser(id: String): ApiResponse<User> {
        return try {
            val user = api.fetchUser(id)
            ApiResponse.Success(user)
        } catch (e: Exception) {
            ApiResponse.Error(
                message = "ইউজার লোড করতে পারিনি",
                code = 500,
                exception = e
            )
        }
    }
}

// UI তে
fun displayUser(response: ApiResponse<User>) {
    when (response) {
        is ApiResponse.Success -> {
            val userName = response.data.name
            showUserName(userName)
        }
        is ApiResponse.Error -> {
            showError(response.message)
            logError(response.exception)
        }
        ApiResponse.Loading -> {
            showLoadingIndicator()
        }
    }
}
```

### 🎯 **উদাহরণ ২: Navigation Events**

```kotlin
sealed class NavigationEvent {
    data class NavigateTo(val route: String) : NavigationEvent()
    data object NavigateBack : NavigationEvent()
    data class NavigateWithResult(
        val route: String,
        val resultKey: String,
        val result: Any
    ) : NavigationEvent()
    
    // Helper functions
    companion object {
        fun toHome() = NavigateTo("home")
        fun toProfile(userId: String) = NavigateTo("profile/$userId")
        fun toSettings() = NavigateTo("settings")
    }
}

// ViewModel এ
class MyViewModel : ViewModel() {
    private val _navigationEvent = MutableSharedFlow<NavigationEvent>()
    val navigationEvent = _navigationEvent.asSharedFlow()
    
    fun onProfileClick(userId: String) {
        viewModelScope.launch {
            _navigationEvent.emit(NavigationEvent.toProfile(userId))
        }
    }
    
    fun onBackPressed() {
        viewModelScope.launch {
            _navigationEvent.emit(NavigationEvent.NavigateBack)
        }
    }
}

// Activity/Fragment এ
lifecycleScope.launch {
    viewModel.navigationEvent.collect { event ->
        when (event) {
            is NavigationEvent.NavigateTo -> {
                navController.navigate(event.route)
            }
            NavigationEvent.NavigateBack -> {
                navController.navigateUp()
            }
            is NavigationEvent.NavigateWithResult -> {
                navController.navigate(event.route)
                navController.currentBackStackEntry
                    ?.savedStateHandle
                    ?.set(event.resultKey, event.result)
            }
        }
    }
}
```

### 🎯 **উদাহরণ ৩: Form Validation**

```kotlin
sealed class ValidationResult {
    data object Valid : ValidationResult()
    data class Invalid(val errors: List<String>) : ValidationResult() {
        constructor(vararg errors: String) : this(errors.toList())
    }
    
    // Utility functions
    fun isValid(): Boolean = this is Valid
    
    fun getErrorsOrEmpty(): List<String> = when (this) {
        is Invalid -> errors
        Valid -> emptyList()
    }
    
    // Combine multiple validations
    operator fun plus(other: ValidationResult): ValidationResult {
        return when {
            this is Valid && other is Valid -> Valid
            this is Invalid && other is Invalid -> Invalid(this.errors + other.errors)
            this is Invalid -> this
            else -> other
        }
    }
}

// Validator class
object Validators {
    fun validateEmail(email: String): ValidationResult {
        return if (email.contains("@") && email.contains(".")) {
            ValidationResult.Valid
        } else {
            ValidationResult.Invalid("ইমেইল সঠিক নয়")
        }
    }
    
    fun validatePassword(password: String): ValidationResult {
        val errors = mutableListOf<String>()
        
        if (password.length < 8) {
            errors.add("পাসওয়ার্ড কমপক্ষে ৮ অক্ষরের হতে হবে")
        }
        if (!password.any { it.isDigit() }) {
            errors.add("পাসওয়ার্ডে অন্তত একটি সংখ্যা থাকতে হবে")
        }
        if (!password.any { it.isUpperCase() }) {
            errors.add("পাসওয়ার্ডে অন্তত একটি বড় হাতের অক্ষর থাকতে হবে")
        }
        
        return if (errors.isEmpty()) {
            ValidationResult.Valid
        } else {
            ValidationResult.Invalid(errors)
        }
    }
    
    fun validatePhoneNumber(phone: String): ValidationResult {
        val bangladeshiPattern = "^(\\+8801|01)[3-9]\\d{8}$"
        return if (phone.matches(bangladeshiPattern.toRegex())) {
            ValidationResult.Valid
        } else {
            ValidationResult.Invalid("বাংলাদেশি ফোন নম্বর সঠিক নয়")
        }
    }
}

// ব্যবহার
class RegistrationViewModel : ViewModel() {
    fun validateRegistrationForm(
        email: String,
        password: String,
        phone: String
    ): ValidationResult {
        val emailValidation = Validators.validateEmail(email)
        val passwordValidation = Validators.validatePassword(password)
        val phoneValidation = Validators.validatePhoneNumber(phone)
        
        return emailValidation + passwordValidation + phoneValidation
    }
    
    fun register(email: String, password: String, phone: String) {
        val validation = validateRegistrationForm(email, password, phone)
        
        when (validation) {
            ValidationResult.Valid -> {
                // Registration proceed করো
                performRegistration(email, password, phone)
            }
            is ValidationResult.Invalid -> {
                // Error গুলো দেখাও
                validation.errors.forEach { error ->
                    showError(error)
                }
            }
        }
    }
}
```

### 🎯 **উদাহরণ ৪: File Operations**

```kotlin
sealed class FileOperation {
    data class Read(val path: String) : FileOperation()
    data class Write(val path: String, val content: String) : FileOperation()
    data class Delete(val path: String) : FileOperation()
    data class Copy(val source: String, val destination: String) : FileOperation()
    
    // Execute করার function
    suspend fun execute(): FileOperationResult {
        return when (this) {
            is Read -> {
                try {
                    val content = readFile(path)
                    FileOperationResult.Success("ফাইল পড়া হয়েছে")
                } catch (e: Exception) {
                    FileOperationResult.Failure(e.message ?: "ফাইল পড়া যায়নি")
                }
            }
            is Write -> {
                try {
                    writeFile(path, content)
                    FileOperationResult.Success("ফাইল লেখা হয়েছে")
                } catch (e: Exception) {
                    FileOperationResult.Failure(e.message ?: "ফাইল লেখা যায়নি")
                }
            }
            is Delete -> {
                try {
                    deleteFile(path)
                    FileOperationResult.Success("ফাইল মুছে ফেলা হয়েছে")
                } catch (e: Exception) {
                    FileOperationResult.Failure(e.message ?: "ফাইল মুছা যায়নি")
                }
            }
            is Copy -> {
                try {
                    copyFile(source, destination)
                    FileOperationResult.Success("ফাইল কপি হয়েছে")
                } catch (e: Exception) {
                    FileOperationResult.Failure(e.message ?: "ফাইল কপি করা যায়নি")
                }
            }
        }
    }
}

sealed class FileOperationResult {
    data class Success(val message: String) : FileOperationResult()
    data class Failure(val error: String) : FileOperationResult()
}
```

---

## Best Practices

### ✅ **১. Data Object ব্যবহার করুন (Kotlin 1.9+)**
```kotlin
sealed class LoadingState {
    data object Loading : LoadingState()  // ✅ সঠিক
    object Loaded : LoadingState()        // ⚠️ পুরনো পদ্ধতি
}
```

### ✅ **২. Generic Type ব্যবহার করুন**
```kotlin
sealed class Resource<out T> {
    data class Success<T>(val data: T) : Resource<T>()
    data class Error(val exception: Throwable) : Resource<Nothing>()
    data object Loading : Resource<Nothing>()
}
```

### ✅ **৩. Companion Object এ Helper Functions**
```kotlin
sealed class UiEvent {
    data class ShowToast(val message: String) : UiEvent()
    data class Navigate(val route: String) : UiEvent()
    
    companion object {
        fun success(message: String) = ShowToast(message)
        fun error(message: String) = ShowToast("এরর: $message")
        fun goToHome() = Navigate("home")
    }
}
```

### ✅ **৪. Extension Functions ব্যবহার**
```kotlin
sealed class Result<out T> {
    data class Success<T>(val data: T) : Result<T>()
    data class Error(val message: String) : Result<Nothing>()
}

// Extension functions
fun <T> Result<T>.onSuccess(action: (T) -> Unit): Result<T> {
    if (this is Result.Success) action(data)
    return this
}

fun <T> Result<T>.onError(action: (String) -> Unit): Result<T> {
    if (this is Result.Error) action(message)
    return this
}

// ব্যবহার
getUserData()
    .onSuccess { user -> println("ইউজার: ${user.name}") }
    .onError { error -> println("এরর: $error") }
```

### ✅ **৫. Sealed Class Hierarchy**
```kotlin
sealed class Animal {
    abstract val name: String
    abstract fun makeSound(): String
    
    sealed class Mammal : Animal() {
        data class Dog(override val name: String) : Mammal() {
            override fun makeSound() = "ঘেউ ঘেউ"
        }
        data class Cat(override val name: String) : Mammal() {
            override fun makeSound() = "মিয়াও"
        }
    }
    
    sealed class Bird : Animal() {
        data class Parrot(override val name: String) : Bird() {
            override fun makeSound() = "টিয়া টিয়া"
        }
    }
}

fun animalSound(animal: Animal): String {
    return when (animal) {
        is Animal.Mammal.Dog -> "${animal.name} বলছে: ${animal.makeSound()}"
        is Animal.Mammal.Cat -> "${animal.name} বলছে: ${animal.makeSound()}"
        is Animal.Bird.Parrot -> "${animal.name} বলছে: ${animal.makeSound()}"
    }
}
```

---

## সারাংশ

### 📌 **মনে রাখার পয়েন্ট:**

1. **Sealed Class = সীমিত সংখ্যক subclass** যা compile-time এ জানা
2. **When expression = exhaustive** (else লাগে না)
3. **Enum থেকে ভালো** = প্রতিটি subclass আলাদা data রাখতে পারে
4. **State management** এর জন্য পারফেক্ট
5. **Type-safe** এবং **null-safe**

### 🎓 **কখন কোনটা?**

- **Sealed Class**: Common behavior + single inheritance
- **Sealed Interface**: Multiple inheritance + flexibility
- **Enum**: শুধু constants, কোনো data নেই
- **Open Class**: Unlimited subclasses দরকার হলে

### 💡 **Android Development এ প্রয়োগ:**

- ✅ UI State management
- ✅ API Response handling
- ✅ Navigation events
- ✅ Form validation
- ✅ Error handling
- ✅ Network status

---

## চূড়ান্ত উদাহরণ - পূর্ণাঙ্গ Android App

```kotlin
// Domain Layer - Sealed Classes
sealed class AuthState {
    data object Unauthenticated : AuthState()
    data class Authenticated(val user: User) : AuthState()
    data object Loading : AuthState()
    data class Error(val message: String) : AuthState()
}

sealed class AuthEvent {
    data class Login(val email: String, val password: String) : AuthEvent()
    data object Logout : AuthEvent()
    data class Register(val email: String, val password: String, val name: String) : AuthEvent()
}

// ViewModel
class AuthViewModel : ViewModel() {
    private val _authState = MutableStateFlow<AuthState>(AuthState.Unauthenticated)
    val authState: StateFlow<AuthState> = _authState
    
    fun handleEvent(event: AuthEvent) {
        when (event) {
            is AuthEvent.Login -> login(event.email, event.password)
            is AuthEvent.Logout -> logout()
            is AuthEvent.Register -> register(event.email, event.password, event.name)
        }
    }
    
    private fun login(email: String, password: String) {
        viewModelScope.launch {
            _authState.value = AuthState.Loading
            
            val validation = Validators.validateEmail(email) + 
                           Validators.validatePassword(password)
            
            when (validation) {
                ValidationResult.Valid -> {
                    try {
                        val user = authRepository.login(email, password)
                        _authState.value = AuthState.Authenticated(user)
                    } catch (e: Exception) {
                        _authState.value = AuthState.Error("লগইন ব্যর্থ: ${e.message}")
                    }
                }
                is ValidationResult.Invalid -> {
                    _authState.value = AuthState.Error(
                        validation.errors.joinToString("\n")
                    )
                }
            }
        }
    }
    
    private fun logout() {
        viewModelScope.launch {
            authRepository.logout()
            _authState.value = AuthState.Unauthenticated
        }
    }
    
    private fun register(email: String, password: String, name: String) {
        // Similar implementation
    }
}

// Composable UI
@Composable
fun AuthScreen(viewModel: AuthViewModel = hiltViewModel()) {
    val authState by viewModel.authState.collectAsState()
    
    when (val state = authState) {
        AuthState.Unauthenticated -> {
            LoginForm(
                onLogin = { email, password ->
                    viewModel.handleEvent(AuthEvent.Login(email, password))
                }
            )
        }
        is AuthState.Authenticated -> {
            HomeScreen(user = state.user)
        }
        AuthState.Loading -> {
            LoadingIndicator()
        }
        is AuthState.Error -> {
            ErrorMessage(message = state.message)
        }
    }
}
```

---

## উপসংহার

Sealed Class হলো Kotlin এর একটি শক্তিশালী feature যা আপনার কোডকে **type-safe**, **readable**, এবং **maintainable** করে তোলে। Android Development এ বিশেষ করে **State Management** এর জন্য এটি অপরিহার্য।

**মনে রাখবেন:** 
- Sealed Class = নিয়ন্ত্রিত inheritance
- When expression = exhaustive checking
- Type safety = কম্পাইলার আপনার বন্ধু!

**Happy Coding! 🚀**

---

*এই টিউটোরিয়ালটি তৈরি করেছে Claude - আপনার Android Development যাত্রায় শুভকামনা রইল!*
