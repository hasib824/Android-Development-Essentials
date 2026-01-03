# Kotlin Lambda Expression - Complete Interview Guide

## 📚 Table of Contents
1. [Lambda Expression কী এবং কেন ব্যবহার করবেন](#lambda-expression-কী-এবং-কেন-ব্যবহার-করবেন)
2. [Lambda এর Syntax এবং Structure](#lambda-এর-syntax-এবং-structure)
3. [Lambda vs Anonymous Function vs Regular Function](#lambda-vs-anonymous-function-vs-regular-function)
4. [Higher-Order Functions](#higher-order-functions)
5. [Closures এবং Variable Capture](#closures-এবং-variable-capture)
6. [Trailing Lambda এবং DSL](#trailing-lambda-এবং-dsl)
7. [Collection Operations with Lambda](#collection-operations-with-lambda)
8. [Android Development এ Lambda](#android-development-এ-lambda)
9. [Performance Considerations](#performance-considerations)
10. [Common Interview Questions এবং Answers](#common-interview-questions-এবং-answers)

---

## Lambda Expression কী এবং কেন ব্যবহার করবেন

### Definition
Lambda expression হলো একটি **anonymous function** যা আপনি variable এর মতো treat করতে পারবেন। এটি function literal হিসেবেও পরিচিত।

### কেন Lambda ব্যবহার করবেন?
```kotlin
// ❌ Traditional Way - বেশি boilerplate code
button.setOnClickListener(object : View.OnClickListener {
    override fun onClick(v: View?) {
        Toast.makeText(context, "Clicked", Toast.LENGTH_SHORT).show()
    }
})

// ✅ Lambda Way - Clean এবং Concise
button.setOnClickListener { 
    Toast.makeText(context, "Clicked", Toast.LENGTH_SHORT).show()
}
```

**সুবিধা:**
- কম boilerplate code
- বেশি readable এবং maintainable
- Functional programming support
- Modern Android development এর standard

---

## Lambda এর Syntax এবং Structure

### Basic Syntax
```kotlin
val lambdaName: (ParameterType1, ParameterType2) -> ReturnType = { param1, param2 ->
    // function body
    result // last expression হলো return value
}
```

### বিভিন্ন Format

#### 1. No Parameter Lambda
```kotlin
// Type inference সহ
val greeting = { println("Hello World!") }

// Explicit type সহ
val greetingExplicit: () -> Unit = { println("Hello World!") }

// Call করা
greeting()  // Output: Hello World!
```

#### 2. Single Parameter Lambda
```kotlin
// 'it' keyword ব্যবহার করে (implicit parameter name)
val square = { number: Int -> number * number }
val squareWithIt: (Int) -> Int = { it * it }

println(square(5))      // 25
println(squareWithIt(5)) // 25

// List এর সাথে
val numbers = listOf(1, 2, 3, 4, 5)
val doubled = numbers.map { it * 2 }
println(doubled)  // [2, 4, 6, 8, 10]
```

#### 3. Multiple Parameters Lambda
```kotlin
// Explicit parameter names
val sum = { a: Int, b: Int -> a + b }

// Type inference
val multiply: (Int, Int) -> Int = { x, y -> x * y }

// 3 parameters
val calculate: (Int, Int, String) -> Int = { a, b, operation ->
    when(operation) {
        "+" -> a + b
        "-" -> a - b
        "*" -> a * b
        "/" -> if (b != 0) a / b else 0
        else -> 0
    }
}

println(sum(5, 3))              // 8
println(multiply(4, 5))         // 20
println(calculate(10, 2, "*"))  // 20
```

#### 4. Lambda with Return Type
```kotlin
// Simple return
val isEven: (Int) -> Boolean = { it % 2 == 0 }

// Complex logic with implicit return
val findGrade: (Int) -> String = { marks ->
    when {
        marks >= 90 -> "A+"
        marks >= 80 -> "A"
        marks >= 70 -> "B"
        marks >= 60 -> "C"
        else -> "F"
    }
}

// Explicit return (labeled return)
val processData: (List<Int>) -> Int = lambda@ { numbers ->
    if (numbers.isEmpty()) return@lambda 0
    numbers.sum()
}

println(isEven(4))           // true
println(findGrade(85))       // A
println(processData(listOf(1,2,3))) // 6
```

#### 5. Unit Return Type
```kotlin
// Return type Unit (কিছু return করে না)
val printMessage: (String) -> Unit = { message ->
    println("Message: $message")
}

// যখন শুধু side effect করতে হয়
val logger: (String, String) -> Unit = { tag, message ->
    Log.d(tag, message)
}

printMessage("Hello")  // Output: Message: Hello
```

---

## Lambda vs Anonymous Function vs Regular Function

### Comparison Table

| Feature | Regular Function | Anonymous Function | Lambda |
|---------|-----------------|-------------------|---------|
| Name | আছে | নেই | নেই |
| Return keyword | প্রয়োজন | প্রয়োজন | প্রয়োজন নেই (implicit) |
| Syntax | Verbose | Medium | Concise |
| Type inference | Limited | Good | Excellent |

### Code Examples
```kotlin
// 1. Regular Function
fun regularSum(a: Int, b: Int): Int {
    return a + b
}

// 2. Anonymous Function
val anonymousSum = fun(a: Int, b: Int): Int {
    return a + b
}

// 3. Lambda Expression
val lambdaSum = { a: Int, b: Int -> a + b }

// সবগুলোর output same
println(regularSum(5, 3))    // 8
println(anonymousSum(5, 3))  // 8
println(lambdaSum(5, 3))     // 8
```

### When to use what?
```kotlin
// Regular Function - Reusable, complex logic
fun validateUser(user: User): Boolean {
    if (user.name.isEmpty()) return false
    if (user.email.isEmpty()) return false
    if (!user.email.contains("@")) return false
    return true
}

// Anonymous Function - Early return প্রয়োজন হলে
val processItems = fun(items: List<Item>): List<Item> {
    if (items.isEmpty()) return emptyList()
    return items.filter { it.isValid }
}

// Lambda - Short, inline operations
val filterActive = items.filter { it.isActive }
val sortByName = items.sortedBy { it.name }
```

---

## Higher-Order Functions

### Definition
Higher-order function হলো এমন function যা:
1. Lambda/function কে parameter হিসেবে নেয় অথবা
2. Lambda/function return করে

### Examples

#### 1. Function as Parameter
```kotlin
// Higher-order function definition
fun performOperation(a: Int, b: Int, operation: (Int, Int) -> Int): Int {
    return operation(a, b)
}

// বিভিন্ন operation pass করা
val sum = performOperation(10, 5) { x, y -> x + y }
val multiply = performOperation(10, 5) { x, y -> x * y }
val power = performOperation(2, 3) { base, exp -> 
    var result = 1
    repeat(exp) { result *= base }
    result
}

println(sum)      // 15
println(multiply) // 50
println(power)    // 8
```

#### 2. Function Returning Function
```kotlin
// Function যা function return করে
fun createMultiplier(factor: Int): (Int) -> Int {
    return { number -> number * factor }
}

val doubler = createMultiplier(2)
val tripler = createMultiplier(3)

println(doubler(5))  // 10
println(tripler(5))  // 15

// Real-world example: Logger factory
fun createLogger(tag: String): (String) -> Unit {
    return { message ->
        Log.d(tag, message)
    }
}

val networkLogger = createLogger("NETWORK")
val databaseLogger = createLogger("DATABASE")

networkLogger("API call started")     // Log: NETWORK: API call started
databaseLogger("Query executed")      // Log: DATABASE: Query executed
```

#### 3. Multiple Lambda Parameters
```kotlin
fun processUserData(
    user: User,
    onSuccess: (User) -> Unit,
    onError: (String) -> Unit,
    onLoading: () -> Unit
) {
    onLoading()
    
    try {
        // Process user
        if (user.isValid()) {
            onSuccess(user)
        } else {
            onError("Invalid user data")
        }
    } catch (e: Exception) {
        onError(e.message ?: "Unknown error")
    }
}

// Usage
processUserData(
    user = currentUser,
    onSuccess = { user ->
        binding.userName.text = user.name
        binding.userEmail.text = user.email
    },
    onError = { errorMsg ->
        Toast.makeText(this, errorMsg, Toast.LENGTH_SHORT).show()
    },
    onLoading = {
        binding.progressBar.visibility = View.VISIBLE
    }
)
```

---

## Closures এবং Variable Capture

### Closure কী?

Closure হলো lambda এর একটি বৈশিষ্ট্য যেখানে lambda তার surrounding scope এর variables কে access এবং modify করতে পারে।

### Variable Capture Examples

#### 1. Read-only Capture
```kotlin
fun createCounter(): () -> Int {
    var count = 0
    return {
        count++  // Capturing এবং modifying outer variable
        count
    }
}

val counter = createCounter()
println(counter())  // 1
println(counter())  // 2
println(counter())  // 3
```

#### 2. Mutable Variable Capture
```kotlin
fun sumWithLogger(): (Int) -> Int {
    var totalSum = 0
    var callCount = 0
    
    return { number ->
        callCount++
        totalSum += number
        println("Call #$callCount: Added $number, Total: $totalSum")
        totalSum
    }
}

val sumLogger = sumWithLogger()
sumLogger(5)   // Call #1: Added 5, Total: 5
sumLogger(10)  // Call #2: Added 10, Total: 15
sumLogger(3)   // Call #3: Added 3, Total: 18
```

#### 3. Android Example - Event Tracking
```kotlin
class EventTracker {
    private var eventCount = 0
    private val events = mutableListOf<String>()
    
    fun trackEvent(eventName: String, action: () -> Unit) {
        eventCount++
        events.add(eventName)
        println("Event #$eventCount: $eventName")
        action()
    }
    
    fun getEventLambda(): (String) -> Unit {
        return { eventName ->
            eventCount++
            events.add(eventName)
            println("Tracked: $eventName (Total: $eventCount)")
        }
    }
}

// Usage
val tracker = EventTracker()
val quickTrack = tracker.getEventLambda()

quickTrack("Button Clicked")     // Tracked: Button Clicked (Total: 1)
quickTrack("Screen Viewed")      // Tracked: Screen Viewed (Total: 2)
quickTrack("Item Purchased")     // Tracked: Item Purchased (Total: 3)
```

#### 4. Scope Function এর সাথে Closure
```kotlin
class UserManager {
    private var currentUser: User? = null
    private val loginAttempts = mutableMapOf<String, Int>()
    
    fun login(email: String, password: String): Result<User> {
        return runCatching {
            // Closure capturing loginAttempts
            val attempts = loginAttempts.getOrDefault(email, 0)
            
            if (attempts >= 3) {
                throw Exception("Account locked: Too many attempts")
            }
            
            // Simulate authentication
            val user = authenticateUser(email, password)
            
            user?.let {
                currentUser = it
                loginAttempts.remove(email)
                Result.success(it)
            } ?: run {
                loginAttempts[email] = attempts + 1
                throw Exception("Invalid credentials")
            }
        }.getOrElse { 
            Result.failure(it) 
        }
    }
}
```

---

## Trailing Lambda এবং DSL

### Trailing Lambda Convention

যদি function এর **last parameter** একটি lambda হয়, তাহলে আপনি lambda টি parenthesis এর বাইরে লিখতে পারবেন।
```kotlin
// Standard syntax
fun processData(data: String, processor: (String) -> String): String {
    return processor(data)
}

// Normal call
val result1 = processData("hello", { it.uppercase() })

// Trailing lambda (recommended)
val result2 = processData("hello") { it.uppercase() }

// যদি শুধুমাত্র lambda parameter থাকে, parenthesis বাদ দিতে পারবেন
fun execute(action: () -> Unit) {
    action()
}

execute { println("Executed!") }
```

### Multiple Parameters with Trailing Lambda
```kotlin
fun fetchData(
    url: String,
    timeout: Int,
    onComplete: (String) -> Unit
) {
    // Fetch logic
    onComplete("Data from $url")
}

// Trailing lambda syntax
fetchData("https://api.example.com", 5000) { data ->
    println("Received: $data")
}
```

### DSL (Domain Specific Language) Creation
```kotlin
// HTML DSL Example
class HTML {
    private val content = StringBuilder()
    
    fun head(init: Head.() -> Unit) {
        val head = Head()
        head.init()
        content.append(head.toString())
    }
    
    fun body(init: Body.() -> Unit) {
        val body = Body()
        body.init()
        content.append(body.toString())
    }
    
    override fun toString() = "<html>$content</html>"
}

class Head {
    private var title = ""
    
    fun title(text: String) {
        title = text
    }
    
    override fun toString() = "<head><title>$title</title></head>"
}

class Body {
    private val content = StringBuilder()
    
    fun h1(text: String) {
        content.append("<h1>$text</h1>")
    }
    
    fun p(text: String) {
        content.append("<p>$text</p>")
    }
    
    override fun toString() = "<body>$content</body>"
}

// DSL function
fun html(init: HTML.() -> Unit): HTML {
    val html = HTML()
    html.init()
    return html
}

// Usage - দেখুন কত সুন্দর readable!
val webpage = html {
    head {
        title("My Webpage")
    }
    body {
        h1("Welcome to Kotlin")
        p("Lambda expressions are powerful!")
        p("DSL makes code readable.")
    }
}

println(webpage)
```

### Android View DSL Example
```kotlin
// Custom View Builder DSL
class ViewBuilder(private val context: Context) {
    private val views = mutableListOf<View>()
    
    fun textView(init: TextView.() -> Unit) {
        val tv = TextView(context)
        tv.init()
        views.add(tv)
    }
    
    fun button(init: Button.() -> Unit) {
        val btn = Button(context)
        btn.init()
        views.add(btn)
    }
    
    fun build(): List<View> = views
}

fun createViews(context: Context, init: ViewBuilder.() -> Unit): List<View> {
    val builder = ViewBuilder(context)
    builder.init()
    return builder.build()
}

// Usage in Activity
val views = createViews(this) {
    textView {
        text = "Enter your name"
        textSize = 18f
        setTextColor(Color.BLACK)
    }
    
    button {
        text = "Submit"
        setOnClickListener {
            Toast.makeText(context, "Clicked!", Toast.LENGTH_SHORT).show()
        }
    }
}
```

---

## Collection Operations with Lambda

### Standard Collection Functions

#### 1. Transformation Functions
```kotlin
val numbers = listOf(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)

// map - transform করা
val squared = numbers.map { it * it }
println(squared)  // [1, 4, 9, 16, 25, 36, 49, 64, 81, 100]

// mapIndexed - index সহ transform
val indexed = numbers.mapIndexed { index, value -> 
    "Index $index: Value $value" 
}
println(indexed.first())  // Index 0: Value 1

// mapNotNull - null skip করে transform
val strings = listOf("1", "2", "abc", "3", "xyz")
val validNumbers = strings.mapNotNull { it.toIntOrNull() }
println(validNumbers)  // [1, 2, 3]

// flatMap - nested collection flatten করা
val nestedLists = listOf(
    listOf(1, 2, 3),
    listOf(4, 5),
    listOf(6, 7, 8, 9)
)
val flattened = nestedLists.flatMap { it }
println(flattened)  // [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

#### 2. Filtering Functions
```kotlin
val numbers = listOf(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)

// filter - condition match করে
val evenNumbers = numbers.filter { it % 2 == 0 }
println(evenNumbers)  // [2, 4, 6, 8, 10]

// filterNot - condition match করে না
val oddNumbers = numbers.filterNot { it % 2 == 0 }
println(oddNumbers)  // [1, 3, 5, 7, 9]

// filterIndexed - index সহ filter
val skipFirst3 = numbers.filterIndexed { index, _ -> index >= 3 }
println(skipFirst3)  // [4, 5, 6, 7, 8, 9, 10]

// partition - দুই ভাগে ভাগ করা
val (even, odd) = numbers.partition { it % 2 == 0 }
println("Even: $even")  // Even: [2, 4, 6, 8, 10]
println("Odd: $odd")    // Odd: [1, 3, 5, 7, 9]
```

#### 3. Searching Functions
```kotlin
val numbers = listOf(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)

// find - প্রথম match করা element
val firstEven = numbers.find { it % 2 == 0 }
println(firstEven)  // 2

// findLast - শেষ match করা element
val lastEven = numbers.findLast { it % 2 == 0 }
println(lastEven)  // 10

// first - প্রথম match (exception throw করে যদি না থাকে)
val firstGreaterThan5 = numbers.first { it > 5 }
println(firstGreaterThan5)  // 6

// firstOrNull - null return করে যদি না থাকে
val firstGreaterThan20 = numbers.firstOrNull { it > 20 }
println(firstGreaterThan20)  // null

// single - শুধুমাত্র একটি match করা উচিত
val singleDigit5 = listOf(1, 2, 5, 7, 9).single { it == 5 }
println(singleDigit5)  // 5
```

#### 4. Aggregation Functions
```kotlin
val numbers = listOf(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)

// reduce - accumulate করা
val sum = numbers.reduce { acc, value -> acc + value }
println(sum)  // 55

// fold - initial value সহ accumulate
val sumWithInitial = numbers.fold(100) { acc, value -> acc + value }
println(sumWithInitial)  // 155

// sumOf - specific value sum করা
val sumOfSquares = numbers.sumOf { it * it }
println(sumOfSquares)  // 385

// count - condition match করা elements গণনা
val evenCount = numbers.count { it % 2 == 0 }
println(evenCount)  // 5
```

#### 5. Boolean Functions
```kotlin
val numbers = listOf(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)

// any - কোনো একটি match করলে true
val hasEven = numbers.any { it % 2 == 0 }
println(hasEven)  // true

// all - সবগুলো match করলে true
val allPositive = numbers.all { it > 0 }
println(allPositive)  // true

val allEven = numbers.all { it % 2 == 0 }
println(allEven)  // false

// none - কোনোটাই match না করলে true
val noneNegative = numbers.none { it < 0 }
println(noneNegative)  // true
```

#### 6. Grouping এবং Sorting
```kotlin
val words = listOf("apple", "banana", "apricot", "berry", "cherry", "avocado")

// groupBy - key অনুযায়ী group করা
val groupedByFirstLetter = words.groupBy { it.first() }
println(groupedByFirstLetter)
// {a=[apple, apricot, avocado], b=[banana, berry], c=[cherry]}

// sortedBy - sort করা
val sortedByLength = words.sortedBy { it.length }
println(sortedByLength)  // [apple, berry, banana, cherry, apricot, avocado]

// sortedByDescending - descending sort
val sortedDesc = words.sortedByDescending { it.length }
println(sortedDesc)  // [apricot, avocado, banana, cherry, apple, berry]

// associate - map তৈরি করা
val lengthMap = words.associateWith { it.length }
println(lengthMap)  // {apple=5, banana=6, apricot=7, berry=5, cherry=6, avocado=7}
```

### Real-World Android Example
```kotlin
data class Product(
    val id: Int,
    val name: String,
    val price: Double,
    val category: String,
    val inStock: Boolean,
    val rating: Double
)

class ProductRepository {
    
    private val products = listOf(
        Product(1, "Laptop", 50000.0, "Electronics", true, 4.5),
        Product(2, "Phone", 30000.0, "Electronics", true, 4.7),
        Product(3, "Shirt", 500.0, "Clothing", false, 4.0),
        Product(4, "Shoes", 2000.0, "Clothing", true, 4.3),
        Product(5, "Watch", 5000.0, "Accessories", true, 4.6),
        Product(6, "Headphones", 3000.0, "Electronics", true, 4.4)
    )
    
    // Available products only
    fun getAvailableProducts() = products.filter { it.inStock }
    
    // Products by category
    fun getProductsByCategory(category: String) = 
        products.filter { it.category == category && it.inStock }
    
    // Products within price range
    fun getProductsInPriceRange(minPrice: Double, maxPrice: Double) =
        products.filter { it.price in minPrice..maxPrice && it.inStock }
    
    // Top rated products
    fun getTopRatedProducts(minRating: Double) =
        products.filter { it.rating >= minRating && it.inStock }
            .sortedByDescending { it.rating }
    
    // Products grouped by category
    fun getProductsGroupedByCategory() =
        products.filter { it.inStock }
            .groupBy { it.category }
    
    // Calculate statistics
    fun getProductStats() = ProductStats(
        totalProducts = products.count(),
        availableProducts = products.count { it.inStock },
        averagePrice = products.filter { it.inStock }
            .map { it.price }
            .average(),
        averageRating = products.filter { it.inStock }
            .map { it.rating }
            .average(),
        categoryCounts = products.groupBy { it.category }
            .mapValues { it.value.count() }
    )
    
    // Search products
    fun searchProducts(query: String) =
        products.filter { 
            it.name.contains(query, ignoreCase = true) && it.inStock 
        }
}

data class ProductStats(
    val totalProducts: Int,
    val availableProducts: Int,
    val averagePrice: Double,
    val averageRating: Double,
    val categoryCounts: Map<String, Int>
)

// Usage in ViewModel
class ProductViewModel(private val repository: ProductRepository) : ViewModel() {
    
    fun loadProducts() {
        val availableProducts = repository.getAvailableProducts()
        val electronics = repository.getProductsByCategory("Electronics")
        val affordable = repository.getProductsInPriceRange(0.0, 10000.0)
        val topRated = repository.getTopRatedProducts(4.5)
        val grouped = repository.getProductsGroupedByCategory()
        val stats = repository.getProductStats()
    }
}
```

---

## Android Development এ Lambda

### 1. Click Listeners
```kotlin
// View OnClickListener
binding.button.setOnClickListener { view ->
    Toast.makeText(this, "Button clicked!", Toast.LENGTH_SHORT).show()
}

// Multiple click listeners
binding.apply {
    buttonSave.setOnClickListener { saveData() }
    buttonCancel.setOnClickListener { finish() }
    buttonDelete.setOnClickListener { showDeleteDialog() }
}

// Long click listener
binding.itemView.setOnLongClickListener {
    showContextMenu()
    true // return true যদি event consume হয়
}
```

### 2. RecyclerView Adapter
```kotlin
// Adapter with multiple click listeners
class UserAdapter(
    private val onItemClick: (User) -> Unit,
    private val onEditClick: (User) -> Unit,
    private val onDeleteClick: (User) -> Unit
) : ListAdapter<User, UserAdapter.ViewHolder>(UserDiffCallback()) {

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
        val binding = ItemUserBinding.inflate(
            LayoutInflater.from(parent.context), 
            parent, 
            false
        )
        return ViewHolder(binding)
    }

    override fun onBindViewHolder(holder: ViewHolder, position: Int) {
        holder.bind(getItem(position))
    }

    inner class ViewHolder(
        private val binding: ItemUserBinding
    ) : RecyclerView.ViewHolder(binding.root) {

        fun bind(user: User) {
            binding.apply {
                textName.text = user.name
                textEmail.text = user.email
                
                // Item click
                root.setOnClickListener { onItemClick(user) }
                
                // Edit button click
                buttonEdit.setOnClickListener { onEditClick(user) }
                
                // Delete button click
                buttonDelete.setOnClickListener { 
                    onDeleteClick(user) 
                }
            }
        }
    }
}

// Usage in Fragment/Activity
val adapter = UserAdapter(
    onItemClick = { user ->
        navigateToUserDetails(user.id)
    },
    onEditClick = { user ->
        showEditDialog(user)
    },
    onDeleteClick = { user ->
        showDeleteConfirmation(user)
    }
)
```

### 3. LiveData এবং StateFlow Observation
```kotlin
class UserViewModel : ViewModel() {
    
    private val _users = MutableLiveData<List<User>>()
    val users: LiveData<List<User>> = _users
    
    private val _uiState = MutableStateFlow<UiState>(UiState.Loading)
    val uiState: StateFlow<UiState> = _uiState
    
    private val _events = MutableSharedFlow<UserEvent>()
    val events: SharedFlow<UserEvent> = _events
}

// Fragment/Activity এ observe করা
class UserFragment : Fragment() {
    
    private val viewModel: UserViewModel by viewModels()
    
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        
        // LiveData observe
        viewModel.users.observe(viewLifecycleOwner) { users ->
            adapter.submitList(users)
            binding.emptyView.isVisible = users.isEmpty()
        }
        
        // StateFlow collect
        viewLifecycleOwner.lifecycleScope.launch {
            viewModel.uiState.collect { state ->
                when (state) {
                    is UiState.Loading -> showLoading()
                    is UiState.Success -> showSuccess(state.data)
                    is UiState.Error -> showError(state.message)
                }
            }
        }
        
        // SharedFlow collect (events)
        viewLifecycleOwner.lifecycleScope.launch {
            viewModel.events.collect { event ->
                when (event) {
                    is UserEvent.ShowToast -> {
                        Toast.makeText(requireContext(), event.message, Toast.LENGTH_SHORT).show()
                    }
                    is UserEvent.NavigateToDetails -> {
                        findNavController().navigate(
                            R.id.action_to_details,
                            bundleOf("userId" to event.userId)
                        )
                    }
                }
            }
        }
    }
}
```

### 4. Coroutines এবং Async Operations
```kotlin
class UserRepository(
    private val apiService: ApiService,
    private val database: UserDatabase
) {
    
    // Callback style with lambda
    suspend fun getUser(
        userId: Int,
        onSuccess: (User) -> Unit,
        onError: (String) -> Unit
    ) {
        try {
            val user = apiService.getUser(userId)
            database.userDao().insert(user)
            onSuccess(user)
        } catch (e: Exception) {
            onError(e.message ?: "Unknown error")
        }
    }
    
    // Result wrapper style
    suspend fun getUserResult(userId: Int): Result<User> {
        return runCatching {
            apiService.getUser(userId).also { user ->
                database.userDao().insert(user)
            }
        }
    }
    
    // Flow style
    fun getUserFlow(userId: Int): Flow<Result<User>> = flow {
        emit(Result.Loading)
        try {
            val user = apiService.getUser(userId)
            database.userDao().insert(user)
            emit(Result.Success(user))
        } catch (e: Exception) {
            emit(Result.Error(e.message ?: "Unknown error"))
        }
    }
}

// ViewModel এ ব্যবহার
class UserViewModel(private val repository: UserRepository) : ViewModel() {
    
    // Callback style
    fun loadUserCallback(userId: Int) {
        viewModelScope.launch {
            repository.getUser(
                userId = userId,
                onSuccess = { user ->
                    _uiState.value = UiState.Success(user)
                },
                onError = { error ->
                    _uiState.value = UiState.Error(error)
                }
            )
        }
    }
    
    // Result style
    fun loadUserResult(userId: Int) {
        viewModelScope.launch {
            repository.getUserResult(userId)
                .onSuccess { user ->
                    _uiState.value = UiState.Success(user)
                }
                .onFailure { error ->
                    _uiState.value = UiState.Error(error.message ?: "Unknown error")
                }
        }
    }
    
    // Flow style
    fun loadUserFlow(userId: Int) {
        viewModelScope.launch {
            repository.getUserFlow(userId).collect { result ->
                _uiState.value = when (result) {
                    is Result.Loading -> UiState.Loading
                    is Result.Success -> UiState.Success(result.data)
                    is Result.Error -> UiState.Error(result.message)
                }
            }
        }
    }
}
```

### 5. Jetpack Compose
```kotlin
@Composable
fun UserScreen(
    viewModel: UserViewModel = viewModel(),
    onNavigateToDetails: (Int) -> Unit
) {
    val uiState by viewModel.uiState.collectAsState()
    val users by viewModel.users.collectAsState(initial = emptyList())
    
    // Lambda for LaunchedEffect
    LaunchedEffect(Unit) {
        viewModel.loadUsers()
    }
    
    Column(modifier = Modifier.fillMaxSize()) {
        
        // Search bar with lambda
        SearchBar(
            query = viewModel.searchQuery,
            onQueryChange = { query ->
                viewModel.updateSearchQuery(query)
            },
            onSearch = { query ->
                viewModel.searchUsers(query)
            }
        )
        
        when (uiState) {
            is UiState.Loading -> LoadingScreen()
            
            is UiState.Success -> {
                // LazyColumn with lambda
                LazyColumn {
                    items(
                        items = users,
                        key = { it.id }
                    ) { user ->
                        UserItem(
                            user = user,
                            onClick = { onNavigateToDetails(user.id) },
                            onEdit = { viewModel.editUser(user) },
                            onDelete = { viewModel.deleteUser(user) }
                        )
                    }
                }
            }
            
            is UiState.Error -> {
                ErrorScreen(
                    message = (uiState as UiState.Error).message,
                    onRetry = { viewModel.loadUsers() }
                )
            }
        }
    }
}

@Composable
fun UserItem(
    user: User,
    onClick: () -> Unit,
    onEdit: () -> Unit,
    onDelete: () -> Unit
) {
    Card(
        modifier = Modifier
            .fillMaxWidth()
            .padding(8.dp)
            .clickable(onClick = onClick)
    ) {
        Row(
            modifier = Modifier
                .fillMaxWidth()
                .padding(16.dp),
            horizontalArrangement = Arrangement.SpaceBetween
        ) {
            Column {
                Text(text = user.name, style = MaterialTheme.typography.titleMedium)
                Text(text = user.email, style = MaterialTheme.typography.bodySmall)
            }
            
            Row {
                IconButton(onClick = onEdit) {
                    Icon(Icons.Default.Edit, contentDescription = "Edit")
                }
                IconButton(onClick = onDelete) {
                    Icon(Icons.Default.Delete, contentDescription = "Delete")
                }
            }
        }
    }
}
```

### 6. Room Database
```kotlin
@Dao
interface UserDao {
    
    @Query("SELECT * FROM users")
    fun getAllUsers(): Flow<List<User>>
    
    @Query("SELECT * FROM users WHERE id = :userId")
    suspend fun getUserById(userId: Int): User?
    
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertUser(user: User)
    
    @Delete
    suspend fun deleteUser(user: User)
}

// Repository এ ব্যবহার
class UserRepository(private val userDao: UserDao) {
    
    // Flow observe করা
    val allUsers: Flow<List<User>> = userDao.getAllUsers()
    
    // Transformation with lambda
    val activeUsers: Flow<List<User>> = userDao.getAllUsers()
        .map { users -> users.filter { it.isActive } }
    
    val userCount: Flow<Int> = userDao.getAllUsers()
        .map { it.size }
    
    // Complex transformation
    val usersByCategory: Flow<Map<String, List<User>>> = userDao.getAllUsers()
        .map { users ->
            users.groupBy { it.category }
        }
}
```

### 7. Navigation Component
```kotlin
// NavController extension function with lambda
fun NavController.navigateSafely(
    route: String,
    builder: NavOptionsBuilder.() -> Unit = {}
) {
    try {
        navigate(route) {
            builder()
        }
    } catch (e: Exception) {
        Log.e("Navigation", "Error navigating to $route", e)
    }
}

// Usage
findNavController().navigateSafely("user_details/$userId") {
    popUpTo("home") {
        inclusive = false
    }
    launchSingleTop = true
}

// Composable navigation with lambda
@Composable
fun AppNavigation() {
    val navController = rememberNavController()
    
    NavHost(
        navController = navController,
        startDestination = "home"
    ) {
        composable("home") {
            HomeScreen(
                onNavigateToDetails = { userId ->
                    navController.navigate("details/$userId")
                }
            )
        }
        
        composable(
            route = "details/{userId}",
            arguments = listOf(navArgument("userId") { type = NavType.IntType })
        ) { backStackEntry ->
            val userId = backStackEntry.arguments?.getInt("userId") ?: 0
            DetailsScreen(
                userId = userId,
                onBack = { navController.popBackStack() }
            )
        }
    }
}
```

### 8. WorkManager
```kotlin
class SyncWorker(
    context: Context,
    params: WorkerParameters
) : CoroutineWorker(context, params) {

    override suspend fun doWork(): Result {
        return try {
            // Work with lambda callbacks
            syncData(
                onProgress = { progress ->
                    setProgress(workDataOf("progress" to progress))
                },
                onComplete = { result ->
                    Log.d("SyncWorker", "Sync completed: $result")
                }
            )
            Result.success()
        } catch (e: Exception) {
            Result.failure(workDataOf("error" to e.message))
        }
    }
    
    private suspend fun syncData(
        onProgress: (Int) -> Unit,
        onComplete: (String) -> Unit
    ) {
        // Simulation
        for (i in 0..100 step 10) {
            delay(100)
            onProgress(i)
        }
        onComplete("Sync successful")
    }
}

// Enqueue work with lambda
class SyncManager(private val workManager: WorkManager) {
    
    fun scheduleSync(onEnqueued: (UUID) -> Unit) {
        val syncRequest = OneTimeWorkRequestBuilder<SyncWorker>()
            .setConstraints(
                Constraints.Builder()
                    .setRequiredNetworkType(NetworkType.CONNECTED)
                    .build()
            )
            .build()
        
        workManager.enqueue(syncRequest)
        onEnqueued(syncRequest.id)
        
        // Observe work progress with lambda
        workManager.getWorkInfoByIdLiveData(syncRequest.id)
            .observeForever { workInfo ->
                when (workInfo.state) {
                    WorkInfo.State.RUNNING -> {
                        val progress = workInfo.progress.getInt("progress", 0)
                        Log.d("SyncManager", "Progress: $progress%")
                    }
                    WorkInfo.State.SUCCEEDED -> {
                        Log.d("SyncManager", "Sync completed successfully")
                    }
                    WorkInfo.State.FAILED -> {
                        val error = workInfo.outputData.getString("error")
                        Log.e("SyncManager", "Sync failed: $error")
                    }
                    else -> {}
                }
            }
    }
}
```

---

## Performance Considerations

### 1. Inline Functions

Lambda তৈরি করলে একটি object create হয়, যা memory overhead সৃষ্টি করে। `inline` keyword ব্যবহার করলে এই overhead কমানো যায়।
```kotlin
// ❌ Non-inline function - Lambda object create হবে
fun measureTime(block: () -> Unit): Long {
    val startTime = System.currentTimeMillis()
    block()
    return System.currentTimeMillis() - startTime
}

// ✅ Inline function - No lambda object, code inline হবে
inline fun measureTimeInline(block: () -> Unit): Long {
    val startTime = System.currentTimeMillis()
    block()
    return System.currentTimeMillis() - startTime
}

// Usage
val time1 = measureTime {
    // Complex operation
    (1..1000000).sum()
}

val time2 = measureTimeInline {
    // Complex operation
    (1..1000000).sum()
}

// time2 দ্রুত হবে কারণ lambda object create হয়নি
```

### 2. noinline Parameter

যদি সব lambda parameter কে inline করতে না চান:
```kotlin
inline fun processData(
    data: List<Int>,
    inlineBlock: (Int) -> Int,
    noinline callback: (String) -> Unit
) {
    val result = data.map(inlineBlock)
    callback("Processed ${result.size} items")
}

// Usage
processData(
    data = listOf(1, 2, 3, 4, 5),
    inlineBlock = { it * 2 },          // এটি inline হবে
    callback = { msg ->                 // এটি inline হবে না
        Log.d("Processing", msg)
    }
)
```

### 3. crossinline Keyword

Non-local return prevent করার জন্য:
```kotlin
// ❌ Without crossinline - compilation error
inline fun runInBackground(crossinline task: () -> Unit) {
    Thread {
        task()  // Lambda এ return থাকলে error হবে
    }.start()
}

// Usage
fun example() {
    runInBackground {
        println("Running")
        // return  // এটি error দিবে কারণ crossinline
    }
}
```

### 4. Lambda vs Function Reference
```kotlin
val numbers = listOf(1, 2, 3, 4, 5)

// ❌ Lambda - নতুন lambda object create
val doubled1 = numbers.map { it * 2 }

// ✅ Function reference - efficient
fun double(x: Int) = x * 2
val doubled2 = numbers.map(::double)

// String functions
val names = listOf("hasib", "rahim", "karim")

// Lambda
val upperCase1 = names.map { it.uppercase() }

// Function reference (better)
val upperCase2 = names.map(String::uppercase)
```

### 5. Avoiding Lambda in Loops
```kotlin
// ❌ Bad - Lambda created in every iteration
fun processList(items: List<String>) {
    for (item in items) {
        item.forEach { char ->
            // Lambda created repeatedly
            println(char)
        }
    }
}

// ✅ Good - Lambda created once
fun processListOptimized(items: List<String>) {
    val processor: (Char) -> Unit = { char ->
        println(char)
    }
    
    for (item in items) {
        item.forEach(processor)
    }
}

// ✅ Best - No lambda at all
fun processListBest(items: List<String>) {
    for (item in items) {
        for (char in item) {
            println(char)
        }
    }
}
```

### 6. Memory Leaks with Lambda
```kotlin
// ❌ Potential memory leak
class MyActivity : AppCompatActivity() {
    
    private lateinit var repository: UserRepository
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // Activity reference captured in lambda
        repository.fetchUsers { users ->
            // এখানে Activity এর reference আছে
            // Activity destroy হলেও lambda থাকলে leak হবে
            updateUI(users)
        }
    }
}

// ✅ Solution 1: Using lifecycle-aware components
class MyActivity : AppCompatActivity() {
    
    private val viewModel: UserViewModel by viewModels()
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // ViewModel এ data observe করা - lifecycle aware
        viewModel.users.observe(this) { users ->
            updateUI(users)
        }
    }
}

// ✅ Solution 2: WeakReference ব্যবহার
class UserRepository {
    
    fun fetchUsers(callback: (List<User>) -> Unit) {
        val weakCallback = WeakReference(callback)
        
        // Network call
        apiService.getUsers { users ->
            weakCallback.get()?.invoke(users)
        }
    }
}
```

### 7. Benchmark Example
```kotlin
// Performance comparison
fun performanceComparison() {
    val largeList = (1..1_000_000).toList()
    
    // Lambda with object creation
    val time1 = measureTimeMillis {
        largeList.map { it * 2 }
    }
    
    // Function reference
    fun double(x: Int) = x * 2
    val time2 = measureTimeMillis {
        largeList.map(::double)
    }
    
    // Inline function
    inline fun processInline(list: List<Int>, transform: (Int) -> Int) =
        list.map(transform)
    
    val time3 = measureTimeMillis {
        processInline(largeList) { it * 2 }
    }
    
    println("Lambda: ${time1}ms")
    println("Function reference: ${time2}ms")
    println("Inline: ${time3}ms")
}
```

---

## Common Interview Questions এবং Answers

### Q1: Lambda Expression কী? Regular function থেকে কীভাবে আলাদা?

**Answer:**
Lambda expression হলো anonymous function যা variable এর মতো pass করা যায়।

**পার্থক্য:**

| Feature | Lambda | Regular Function |
|---------|--------|-----------------|
| নাম | নেই | আছে |
| Return keyword | Implicit | Explicit |
| Syntax | Concise | Verbose |
| Reusability | কম | বেশি |
```kotlin
// Regular function
fun add(a: Int, b: Int): Int {
    return a + b
}

// Lambda
val add = { a: Int, b: Int -> a + b }
```

**যখন কোনটা use করবেন:**
- Lambda: One-time use, short operations, functional programming
- Regular function: Complex logic, reusability, named operations

---

### Q2: Higher-Order Function কী? Example দিন।

**Answer:**
Higher-order function হলো এমন function যা:
1. Lambda/function কে parameter হিসেবে নেয় বা
2. Function return করে
```kotlin
// Example 1: Function as parameter
fun calculate(a: Int, b: Int, operation: (Int, Int) -> Int): Int {
    return operation(a, b)
}

val sum = calculate(5, 3) { x, y -> x + y }  // 8
val product = calculate(5, 3) { x, y -> x * y }  // 15

// Example 2: Returning function
fun makeMultiplier(factor: Int): (Int) -> Int {
    return { number -> number * factor }
}

val triple = makeMultiplier(3)
println(triple(5))  // 15

// Android example
fun setupClickListener(button: Button, action: () -> Unit) {
    button.setOnClickListener { action() }
}

setupClickListener(myButton) {
    Toast.makeText(context, "Clicked", Toast.LENGTH_SHORT).show()
}
```

**Real-world use case:**
- Event handlers (onClick, onSuccess, onError)
- Collection operations (map, filter, reduce)
- Dependency injection callbacks
- Repository layer callbacks

---

### Q3: `it` keyword কী? কখন use করবেন?

**Answer:**
`it` হলো single parameter lambda এর implicit parameter name।
```kotlin
// Without 'it' - explicit parameter
val doubled1 = listOf(1, 2, 3).map { number -> number * 2 }

// With 'it' - implicit parameter
val doubled2 = listOf(1, 2, 3).map { it * 2 }

// ভালো practice
// ✅ Good: Simple, clear operation
val squares = numbers.map { it * it }

// ❌ Avoid: Complex nested operations
val result = users.filter { it.age > 18 }
    .map { it.posts.filter { it.likes > 100 } }  // Confusing!

// ✅ Better: Explicit names for nested
val result = users.filter { user -> user.age > 18 }
    .map { user -> 
        user.posts.filter { post -> post.likes > 100 }
    }
```

**কখন use করবেন:**
- Simple, one-line operations
- Parameter এর meaning clear হলে
- Nested lambda না থাকলে

**কখন avoid করবেন:**
- Multiple nested lambdas
- Complex operations
- Readability কমে গেলে

---

### Q4: Trailing Lambda syntax কী? Explain করুন।

**Answer:**
যদি function এর last parameter একটি lambda হয়, তাহলে lambda টি parenthesis এর বাইরে লিখতে পারবেন।
```kotlin
// Normal syntax
fun performTask(name: String, action: () -> Unit) {
    println("Starting $name")
    action()
}

// Standard call
performTask("Download", { println("Downloading...") })

// Trailing lambda (preferred)
performTask("Download") { 
    println("Downloading...") 
}

// যদি শুধু lambda parameter থাকে
fun execute(task: () -> Unit) {
    task()
}

// Parenthesis optional
execute { println("Executed!") }

// Android example
button.setOnClickListener { view ->
    Toast.makeText(context, "Clicked", Toast.LENGTH_SHORT).show()
}

// Retrofit example
apiService.getUsers().enqueue(object : Callback<List<User>> {
    override fun onResponse(call: Call<List<User>>, response: Response<List<User>>) {
        // Handle success
    }
    override fun onFailure(call: Call<List<User>>, t: Throwable) {
        // Handle error
    }
})

// With trailing lambda (using custom wrapper)
apiService.getUsers().enqueue(
    onSuccess = { users -> 
        updateUI(users)
    },
    onError = { error ->
        showError(error)
    }
)
```

**DSL creation:**
```kotlin
// HTML DSL
html {
    head {
        title("My Page")
    }
    body {
        h1("Welcome")
        p("Hello World")
    }
}
```

---

### Q5: Closure কী? Example সহ explain করুন।

**Answer:**
Closure হলো lambda এর ability যেখানে সে তার surrounding scope এর variables কে access এবং modify করতে পারে।
```kotlin
// Example 1: Variable capture
fun createCounter(): () -> Int {
    var count = 0  // Lambda এই variable capture করবে
    return {
        count++
        count
    }
}

val counter = createCounter()
println(counter())  // 1
println(counter())  // 2
println(counter())  // 3

// Example 2: Multiple variables
fun makeCalculator(): Pair<() -> Int, () -> Unit> {
    var total = 0
    var operationCount = 0
    
    val add = { number: Int ->
        total += number
        operationCount++
        total
    }
    
    val reset = {
        total = 0
        operationCount = 0
    }
    
    return Pair(add to reset)
}

// Example 3: Android use case
class EventLogger {
    private var eventCount = 0
    private val events = mutableListOf<String>()
    
    fun createLogger(): (String) -> Unit {
        return { eventName ->
            eventCount++
            events.add(eventName)
            Log.d("Event", "[$eventCount] $eventName")
        }
    }
}

val logger = EventLogger().createLogger()
logger("Button Clicked")   // [1] Button Clicked
logger("Screen Viewed")    // [2] Screen Viewed

// Practical example: Debounce
class SearchManager {
    private var lastSearchTime = 0L
    private val debounceDelay = 300L
    
    fun createDebouncedSearch(): (String) -> Unit {
        return { query ->
            val currentTime = System.currentTimeMillis()
            if (currentTime - lastSearchTime > debounceDelay) {
                lastSearchTime = currentTime
                performSearch(query)
            }
        }
    }
}
```

---

### Q6: `inline`, `noinline`, এবং `crossinline` explain করুন।

**Answer:**

#### `inline`
Function body এবং lambda কে call site এ copy করে performance improve করে।
```kotlin
// Without inline
fun measureTime(block: () -> Unit): Long {
    val start = System.currentTimeMillis()
    block()  // Lambda object created
    return System.currentTimeMillis() - start
}

// With inline
inline fun measureTimeInline(block: () -> Unit): Long {
    val start = System.currentTimeMillis()
    block()  // No lambda object, code inlined
    return System.currentTimeMillis() - start
}
```

**Bytecode difference:**
```kotlin
// Non-inline call
val time = measureTime { println("Hello") }

// Generates:
Function0 block = new Function0() {
    public void invoke() {
        System.out.println("Hello");
    }
};
long time = measureTime(block);

// Inline call
val time = measureTimeInline { println("Hello") }

// Generates:
long start = System.currentTimeMillis();
System.out.println("Hello");
long time = System.currentTimeMillis() - start;
```

#### `noinline`
Specific lambda parameter কে inline করতে না চাইলে।
```kotlin
inline fun processData(
    data: List<Int>,
    transform: (Int) -> Int,        // এটি inline হবে
    noinline logger: (String) -> Unit  // এটি inline হবে না
) {
    val result = data.map(transform)
    logger("Processed ${result.size} items")
}

// কেন noinline দরকার?
// 1. Lambda কে variable এ store করতে হলে
// 2. Lambda কে অন্য function এ pass করতে হলে
// 3. Lambda return করতে হলে

inline fun helper(noinline callback: () -> Unit) {
    val stored = callback  // Only possible with noinline
    anotherFunction(stored)
}
```

#### `crossinline`
Non-local return prevent করার জন্য।
```kotlin
// Problem without crossinline
inline fun runAsync(task: () -> Unit) {
    Thread {
        task()  // Compile error if task() has return
    }.start()
}

fun example() {
    runAsync {
        println("Running")
        return  // Error: 'return' not allowed here
    }
}

// Solution with crossinline
inline fun runAsync(crossinline task: () -> Unit) {
    Thread {
        task()  // OK, return not allowed in task
    }.start()
}

// Real Android example
inline fun executeWithDelay(
    delayMillis: Long,
    crossinline action: () -> Unit
) {
    Handler(Looper.getMainLooper()).postDelayed({
        action()  // Safe, no non-local returns
    }, delayMillis)
}
```

**Summary:**
- `inline`: Performance optimize করে, lambda object create হয় না
- `noinline`: Specific parameter কে inline করতে চাই না
- `crossinline`: Lambda এ return statement prevent করে

---

### Q7: Lambda এবং Function Reference এর মধ্যে পার্থক্য কী?

**Answer:**
```kotlin
val numbers = listOf(1, 2, 3, 4, 5)

// 1. Lambda expression - নতুন object create
val doubled1 = numbers.map { it * 2 }

// 2. Function reference - existing function use
fun double(x: Int) = x * 2
val doubled2 = numbers.map(::double)  // More efficient

// Different types of function references:

// Class member reference
class Calculator {
    fun multiply(x: Int, y: Int) = x * y
}

val calc = Calculator()
val operation: (Int, Int) -> Int = calc::multiply

// Static/top-level function reference
fun isEven(x: Int) = x % 2 == 0
val evenNumbers = numbers.filter(::isEven)

// Constructor reference
data class User(val name: String, val age: Int)
val names = listOf("Alice", "Bob")
val users = names.map { User(it, 25) }          // Lambda
val usersRef = names.map(::User.partially(25))  // Reference (hypothetical)

// Extension function reference
fun String.isPalindrome() = this == this.reversed()
val words = listOf("level", "hello", "radar")
val palindromes = words.filter(String::isPalindrome)

// Android example
// Lambda
binding.button.setOnClickListener { view -> handleClick(view) }

// Method reference
binding.button.setOnClickListener(this::handleClick)

// Property reference
val users = listOf(User("Alice", 25), User("Bob", 30))
val names = users.map { it.name }        // Lambda
val namesRef = users.map(User::name)     // Property reference
```

**কখন কোনটা use করবেন:**
- Lambda: Complex operations, multiple statements
- Function reference: Single function call, better performance

---

### Q8: Lambda এ return statement কীভাবে কাজ করে?

**Answer:**
```kotlin
// 1. Implicit return (normal lambda)
val sum = { a: Int, b: Int -> a + b }  // Last expression returned
println(sum(5, 3))  // 8

// 2. Labeled return (lambda থেকে specific return)
fun processNumbers(numbers: List<Int>) {
    numbers.forEach lambda@{ number ->
        if (number < 0) return@lambda  // শুধু lambda থেকে return
        println(number)
    }
    println("Processing complete")  // এটি execute হবে
}

processNumbers(listOf(1, -2, 3, -4, 5))
// Output:
// 1
// 3
// 5
// Processing complete

// 3. Non-local return (enclosing function থেকে return)
inline fun processInline(numbers: List<Int>, action: (Int) -> Unit) {
    numbers.forEach(action)
}

fun findFirst(numbers: List<Int>): Int {
    processInline(numbers) { number ->
        if (number > 5) {
            return number  // findFirst function থেকে return হবে
        }
    }
    return -1
}

println(findFirst(listOf(1, 2, 8, 3, 9)))  // 8

// 4. Anonymous function এ return (always local)
val numbers = listOf(1, 2, 3, 4, 5)

// Lambda with labeled return
numbers.forEach label@{
    if (it == 3) return@label
    println(it)
}

// Anonymous function with return
numbers.forEach(fun(number) {
    if (number == 3) return  // শুধু anonymous function থেকে return
    println(number)
})

// 5. Android example
fun loadData(callback: (Result) -> Unit) {
    viewModelScope.launch {
        try {
            val data = repository.fetchData()
            callback(Result.Success(data))
            return@launch  // Coroutine থেকে return
        } catch (e: Exception) {
            callback(Result.Error(e.message))
            return@launch
        }
    }
}

// 6. Early return pattern
fun validateUser(user: User): Boolean {
    return user.name.isNotEmpty() &&
           user.email.contains("@") &&
           user.age >= 18
}

// With lambda
fun validateUserLambda(user: User): Boolean = runCatching {
    if (user.name.isEmpty()) return@runCatching false
    if (!user.email.contains("@")) return@runCatching false
    if (user.age < 18) return@runCatching false
    true
}.getOrDefault(false)
```

**Key points:**
- Lambda: Implicit return (last expression)
- `return@label`: Lambda specific return
- `return`: Non-local return (inline function এ)
- Anonymous function: Always local return

---

### Q9: Collection operations (map, filter, reduce) explain করুন।

**Answer:**

#### `map` - Transform each element
```kotlin
val numbers = listOf(1, 2, 3, 4, 5)

// Simple transformation
val doubled = numbers.map { it * 2 }
println(doubled)  // [2, 4, 6, 8, 10]

// Complex transformation
data class User(val id: Int, val name: String)
val users = listOf(
    User(1, "Alice"),
    User(2, "Bob")
)
val names = users.map { it.name }
println(names)  // [Alice, Bob]

// mapIndexed - with index
val indexed = numbers.mapIndexed { index, value ->
    "[$index] = $value"
}
println(indexed)  // [[0] = 1, [1] = 2, ...]

// mapNotNull - skip nulls
val strings = listOf("1", "2", "abc", "3")
val validNumbers = strings.mapNotNull { it.toIntOrNull() }
println(validNumbers)  // [1, 2, 3]
```

#### `filter` - Select elements
```kotlin
val numbers = listOf(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)

// Basic filter
val evenNumbers = numbers.filter { it % 2 == 0 }
println(evenNumbers)  // [2, 4, 6, 8, 10]

// Multiple conditions
val filtered = numbers.filter { it > 3 && it < 8 }
println(filtered)  // [4, 5, 6, 7]

// filterNot - opposite condition
val oddNumbers = numbers.filterNot { it % 2 == 0 }
println(oddNumbers)  // [1, 3, 5, 7, 9]

// filterIndexed - with index
val skipFirst3 = numbers.filterIndexed { index, _ -> 
    index >= 3 
}
println(skipFirst3)  // [4, 5, 6, 7, 8, 9, 10]

// Android example
val activeUsers = users.filter { it.isActive && it.isPremium }
```

#### `reduce` - Accumulate values
```kotlin
val numbers = listOf(1, 2, 3, 4, 5)

// Sum using reduce
val sum = numbers.reduce { acc, value -> acc + value }
println(sum)  // 15

// Product
val product = numbers.reduce { acc, value -> acc * value }
println(product)  // 120

// fold - with initial value
val sumWith100 = numbers.fold(100) { acc, value -> acc + value }
println(sumWith100)  // 115

// Complex example - concatenate strings
val words = listOf("Hello", "Kotlin", "World")
val sentence = words.reduce { acc, word -> "$acc $word" }
println(sentence)  // Hello Kotlin World

// reduceIndexed
val indexed = numbers.reduceIndexed { index, acc, value ->
    acc + (value * index)
}
println(indexed)  // 0 + 2 + 6 + 12 + 20 = 40

// Android example - calculate total price
data class Product(val name: String, val price: Double)
val products = listOf(
    Product("Laptop", 50000.0),
    Product("Phone", 30000.0),
    Product("Tablet", 20000.0)
)

val totalPrice = products.fold(0.0) { total, product ->
    total + product.price
}
println("Total: $totalPrice")  // Total: 100000.0
```

#### Chaining operations
```kotlin
val numbers = listOf(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)

// Complex chain
val result = numbers
    .filter { it % 2 == 0 }           // [2, 4, 6, 8, 10]
    .map { it * it }                   // [4, 16, 36, 64, 100]
    .filter { it > 20 }                // [36, 64, 100]
    .reduce { acc, value -> acc + value }  // 200

println(result)  // 200

// Android example - process users
data class User(
    val id: Int,
    val name: String,
    val age: Int,
    val isPremium: Boolean
)

val users = listOf(/* ... */)

val premiumAdultNames = users
    .filter { it.isPremium }
    .filter { it.age >= 18 }
    .map { it.name }
    .sorted()

println(premiumAdultNames)
```

---

### Q10: Lambda এ memory leak কীভাবে হয় এবং কীভাবে prevent করবেন?

**Answer:**

#### Problem: Activity/Fragment reference capture
```kotlin
// ❌ Memory leak - Activity reference captured
class MainActivity : AppCompatActivity() {
    
    private lateinit var repository: UserRepository
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // Lambda captures 'this' (Activity reference)
        repository.loadUsers { users ->
            // Activity might be destroyed but lambda still holds reference
            updateUI(users)  // Using 'this' implicitly
        }
    }
    
    private fun updateUI(users: List<User>) {
        // Update UI
    }
}

// Problem: Repository holds callback longer than Activity lifecycle
class UserRepository {
    private var callback: ((List<User>) -> Unit)? = null
    
    fun loadUsers(onComplete: (List<User>) -> Unit) {
        callback = onComplete  // Holding reference
        
        // Long running task
        Thread {
            Thread.sleep(10000)
            callback?.invoke(emptyList())
        }.start()
    }
}
```

#### Solution 1: ViewModel + LiveData/StateFlow (Recommended)
```kotlin
// ✅ Proper way - Using ViewModel
class UserViewModel(
    private val repository: UserRepository
) : ViewModel() {
    
    private val _users = MutableLiveData<List<User>>()
    val users: LiveData<List<User>> = _users
    
    fun loadUsers() {
        viewModelScope.launch {
            val result = repository.getUsers()
            _users.value = result
        }
    }
}

// Activity
class MainActivity : AppCompatActivity() {
    
    private val viewModel: UserViewModel by viewModels()
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // Lifecycle aware - automatically cleaned up
        viewModel.users.observe(this) { users ->
            updateUI(users)
        }
        
        viewModel.loadUsers()
    }
}
```

#### Solution 2: WeakReference
```kotlin
// ✅ Using WeakReference
class UserRepository {
    
    fun loadUsers(callback: (List<User>) -> Unit) {
        val weakCallback = WeakReference(callback)
        
        Thread {
            Thread.sleep(5000)
            // Check if callback still exists
            weakCallback.get()?.invoke(emptyList())
        }.start()
    }
}

// Usage
repository.loadUsers { users ->
    updateUI(users)
}
```

#### Solution 3: Manual cleanup
```kotlin
// ✅ Manual cleanup
class MainActivity : AppCompatActivity() {
    
    private val callbacks = mutableListOf<(List<User>) -> Unit>()
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        val callback: (List<User>) -> Unit = { users ->
            updateUI(users)
        }
        
        callbacks.add(callback)
        repository.loadUsers(callback)
    }
    
    override fun onDestroy() {
        super.onDestroy()
        // Clear all callbacks
        callbacks.clear()
        repository.clearCallbacks()
    }
}

class UserRepository {
    private val callbacks = mutableListOf<(List<User>) -> Unit>()
    
    fun loadUsers(callback: (List<User>) -> Unit) {
        callbacks.add(callback)
        // Load users...
    }
    
    fun clearCallbacks() {
        callbacks.clear()
    }
}
```

#### Solution 4: Lifecycle scope
```kotlin
// ✅ Using lifecycleScope
class MainActivity : AppCompatActivity() {
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // Automatically cancelled when Activity destroyed
        lifecycleScope.launch {
            repository.getUsersFlow().collect { users ->
                updateUI(users)
            }
        }
    }
}

class UserRepository {
    fun getUsersFlow(): Flow<List<User>> = flow {
        val users = apiService.getUsers()
        emit(users)
    }
}
```

#### Solution 5: Properly structured callbacks
```kotlin
// ✅ Callback interface with lifecycle
interface UserCallback {
    fun onUsersLoaded(users: List<User>)
    fun isActive(): Boolean
}

class MainActivity : AppCompatActivity(), UserCallback {
    
    private var isActivityActive = false
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        isActivityActive = true
        repository.loadUsers(this)
    }
    
    override fun onUsersLoaded(users: List<User>) {
        if (isActive()) {
            updateUI(users)
        }
    }
    
    override fun isActive(): Boolean = isActivityActive
    
    override fun onDestroy() {
        super.onDestroy()
        isActivityActive = false
    }
}
```

**Best practices:**
1. Always use ViewModel with LiveData/StateFlow for Android
2. Avoid storing Activity/Fragment/View references in callbacks
3. Use WeakReference যদি reference রাখতেই হয়
4. Use lifecycle-aware components (lifecycleScope, viewLifecycleOwner)
5. Clean up manually যদি অন্য কোনো উপায় না থাকে

---

### Q11: SAM (Single Abstract Method) Conversion কী?

**Answer:**
SAM conversion হলো Java interface (যেখানে শুধু একটি abstract method আছে) কে Kotlin lambda দিয়ে replace করার process।
```kotlin
// Java interface with single abstract method
interface OnClickListener {
    fun onClick(view: View)
}

// Java style (verbose)
button.setOnClickListener(object : OnClickListener {
    override fun onClick(view: View) {
        Toast.makeText(context, "Clicked", Toast.LENGTH_SHORT).show()
    }
})

// Kotlin lambda (SAM conversion)
button.setOnClickListener { view ->
    Toast.makeText(context, "Clicked", Toast.LENGTH_SHORT).show()
}

// Even shorter
button.setOnClickListener {
    Toast.makeText(context, "Clicked", Toast.LENGTH_SHORT).show()
}

// More examples:

// Runnable (Java interface)
// Java style
Thread(object : Runnable {
    override fun run() {
        println("Running in thread")
    }
}).start()

// SAM conversion
Thread {
    println("Running in thread")
}.start()

// Comparator
val numbers = listOf(3, 1, 4, 1, 5)

// Java style
val sorted1 = numbers.sortedWith(object : Comparator<Int> {
    override fun compare(a: Int, b: Int): Int {
        return a.compareTo(b)
    }
})

// SAM conversion
val sorted2 = numbers.sortedWith { a, b -> a.compareTo(b) }

// Even simpler
val sorted3 = numbers.sortedWith(Comparator { a, b -> a.compareTo(b) })

// RecyclerView example
class MyAdapter : RecyclerView.Adapter<MyAdapter.ViewHolder>() {
    
    // Java style callback
    interface OnItemClickListener {
        fun onItemClick(position: Int)
    }
    
    private var listener: OnItemClickListener? = null
    
    fun setOnItemClickListener(listener: OnItemClickListener) {
        this.listener = listener
    }
}

// Usage - Java style
adapter.setOnItemClickListener(object : MyAdapter.OnItemClickListener {
    override fun onItemClick(position: Int) {
        handleClick(position)
    }
})

// Better approach - Kotlin style
class MyAdapter(
    private val onItemClick: (Int) -> Unit
) : RecyclerView.Adapter<MyAdapter.ViewHolder>() {
    // ...
}

// Usage - Lambda
val adapter = MyAdapter { position ->
    handleClick(position)
}
```

**Important notes:**
- SAM conversion শুধু Java interfaces এর জন্য
- Kotlin interfaces এর জন্য কাজ করে না (use function type instead)
- Performance: SAM conversion efficient, no overhead
```kotlin
// ❌ Kotlin interface - SAM conversion হবে না
interface KotlinListener {
    fun onEvent()
}

// Use function type instead
typealias EventListener = () -> Unit

class EventManager(private val listener: EventListener) {
    fun triggerEvent() {
        listener()
    }
}

// Usage
val manager = EventManager {
    println("Event triggered")
}
```

---

## 🎯 Interview Preparation Checklist

### Must Know Topics:
- ✅ Lambda basic syntax এবং structure
- ✅ Higher-order functions কী এবং কীভাবে use করে
- ✅ `it` keyword এর proper usage
- ✅ Trailing lambda convention
- ✅ Closure এবং variable capture
- ✅ Return statements in lambda
- ✅ Collection operations (map, filter, reduce)
- ✅ Lambda vs Function reference
- ✅ `inline`, `noinline`, `crossinline` keywords
- ✅ Memory leak prevention
- ✅ SAM conversion

### Android Specific:
- ✅ Click listeners with lambda
- ✅ RecyclerView adapter callbacks
- ✅ LiveData/StateFlow observation
- ✅ Coroutines with lambda
- ✅ Jetpack Compose lambdas
- ✅ Room database queries
- ✅ Navigation with lambda
- ✅ ViewModel callbacks

### Code Practice:
- ✅ Write lambda for different scenarios
- ✅ Convert Java callbacks to Kotlin lambdas
- ✅ Implement higher-order functions
- ✅ Create DSL using lambda
- ✅ Fix memory leak scenarios
- ✅ Optimize lambda performance

### Interview Tips:
1. প্রথমে concept বুঝিয়ে বলুন, তারপর code example দিন
2. Real-world Android example দিতে পারলে ভালো impression হয়
3. Performance implications mention করুন
4. Best practices follow করার কথা বলুন
5. Common mistakes এবং তাদের solutions জানুন

---

## 📝 Practice Questions

### Beginner Level:
1. একটি lambda লিখুন যা দুইটি সংখ্যার বড়টি return করবে
2. List of strings কে uppercase এ convert করুন lambda দিয়ে
3. একটি higher-order function লিখুন যা দুইটি number এবং একটি operation নিবে
4. `it` keyword use করে list filter করুন
5. Trailing lambda syntax ব্যবহার করে একটি function call করুন

### Intermediate Level:
1. Closure concept ব্যবহার করে একটি counter function তৈরি করুন
2. RecyclerView adapter তৈরি করুন multiple lambda callbacks সহ
3. Collection operations chain করে complex data processing করুন
4. Lambda এবং function reference এর performance difference show করুন
5. DSL তৈরি করুন lambda ব্যবহার করে

### Advanced Level:
1. Memory leak scenario তৈরি এবং fix করুন
2. `inline`, `noinline`, `crossinline` এর practical use case demonstrate করুন
3. Custom scope function তৈরি করুন
4. Type-safe builder pattern implement করুন
5. Coroutines এর সাথে complex lambda callback handling

---

## 🎓 Best Practices Summary

### DO:
✅ Lambda ছোট এবং focused রাখুন
✅ Meaningful parameter names ব্যবহার করুন nested lambdas এ
✅ Trailing lambda syntax ব্যবহার করুন
✅ Function reference use করুন যখন possible
✅ `inline` keyword use করুন performance-critical code এ
✅ Lifecycle-aware components ব্যবহার করুন Android এ
✅ Explicit return types specify করুন complex lambdas এ

### DON'T:
❌ Lambda এ complex logic রাখবেন না
❌ Multiple nested lambdas এ `it` ব্যবহার করবেন না
❌ Activity/Fragment reference capture করবেন না lambda এ
❌ Lambda তে heavy computation করবেন না
❌ Anonymous functions এ unnecessarily return keyword use করবেন না
❌ Lambda এ side effects unnecessarily create করবেন না

---

## 📚 Additional Resources

### Official Documentation:
- [Kotlin Lambda Documentation](https://kotlinlang.org/docs/lambdas.html)
- [Higher-Order Functions](https://kotlinlang.org/docs/higher-order-functions.html)
- [Inline Functions](https://kotlinlang.org/docs/inline-functions.html)

### Practice Platforms:
- [Kotlin Koans](https://kotlinlang.org/docs/koans.html)
- [LeetCode Kotlin](https://leetcode.com/)
- [HackerRank Kotlin](https://www.hackerrank.com/domains/kotlin)

---

**শেষ কথা:**
Lambda expression Kotlin এর সবচেয়ে powerful features এর একটি। এটি আপনার code কে concise, readable এবং maintainable করে তোলে। Interview এর জন্য শুধু syntax জানলেই হবে না, practical use cases এবং best practices জানা জরুরি। এই guide follow করলে আপনি lambda নিয়ে যেকোনো interview question handle করতে পারবেন।

**Happy Coding! 🚀**
