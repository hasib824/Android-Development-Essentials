# snapshotFlow - সম্পূর্ণ বাংলা Tutorial 🌊

## সূচিপত্র (Table of Contents)

1. [snapshotFlow কী?](#1-snapshotflow-কী)
2. [কেন snapshotFlow দরকার?](#2-কেন-snapshotflow-দরকার)
3. [Basic Syntax](#3-basic-syntax)
4. [Real-World Use Cases](#4-real-world-use-cases)
5. [Flow Operators with snapshotFlow](#5-flow-operators-with-snapshotflow)
6. [LaunchedEffect vs snapshotFlow](#6-launchedeffect-vs-snapshotflow)
7. [Best Practices](#7-best-practices)
8. [Complete Examples](#8-complete-examples)

---

## 1. snapshotFlow কী?

### 1.1 সংজ্ঞা

**snapshotFlow** হলো একটি Compose API যা **Compose State** কে **Kotlin Flow** এ convert করে।

```kotlin
State → snapshotFlow → Flow → Collect → Side Effect
```

### 1.2 Real Life Analogy 🏪

#### গল্প: দোকানের Stock Monitor

একটা দোকান আছে যেখানে জিনিসপত্র বিক্রি হয়:

```
দোকানে Stock আছে: আম = 100টা

আপনি একজন Manager:
- Stock কমে গেলে alert দিতে হবে
- Stock < 10 হলে → SMS পাঠাও
- Stock = 0 হলে → Email পাঠাও
- কিন্তু প্রতিটা change এ alert না, শুধু important change এ!
```

**এটাই snapshotFlow এর কাজ!** State changes observe করে smart decisions নেওয়া।

---

## 2. কেন snapshotFlow দরকার?

### 2.1 সমস্যা: LaunchedEffect এর সীমাবদ্ধতা

#### ❌ Without snapshotFlow (সমস্যা)

```kotlin
@Composable
fun SearchScreen() {
    var searchText by remember { mutableStateOf("") }
    
    // ❌ প্রতি keystroke এ API call - ভুল!
    LaunchedEffect(searchText) {
        if (searchText.isNotEmpty()) {
            searchAPI(searchText) // Too many calls!
        }
    }
    
    TextField(
        value = searchText,
        onValueChange = { searchText = it }
    )
}
```

**Console Output:**
```
User typing: "H" → API call
User typing: "Ha" → API call
User typing: "Has" → API call
User typing: "Hasi" → API call

❌ 4 API calls! Server overload!
```

#### ✅ With snapshotFlow (সমাধান)

```kotlin
@Composable
fun SearchScreen() {
    var searchText by remember { mutableStateOf("") }
    var results by remember { mutableStateOf<List<String>>(emptyList()) }
    
    // ✅ Debounce দিয়ে optimize করা
    LaunchedEffect(Unit) {
        snapshotFlow { searchText }
            .debounce(500) // 500ms wait
            .filter { it.length >= 3 }
            .distinctUntilChanged()
            .collect { query ->
                println("🔍 Searching for: $query")
                results = searchAPI(query)
            }
    }
    
    Column {
        TextField(
            value = searchText,
            onValueChange = { searchText = it },
            label = { Text("Search") }
        )
        
        results.forEach { result ->
            Text(result)
        }
    }
}
```

**এখন Console Output:**
```
User typing: "H" → (wait...)
User typing: "Ha" → (wait...)
User typing: "Has" → (wait...)
User typing: "Hasi" → (wait 500ms...)
🔍 Searching for: Hasi

✅ শুধু 1 API call! Perfect! 🎯
```

### 2.2 snapshotFlow এর সুবিধা

| Feature | LaunchedEffect | snapshotFlow |
|---------|---------------|--------------|
| **Simple side effect** | ✅ Perfect | ❌ Overkill |
| **State observe** | ⚠️ Limited | ✅ Perfect |
| **Debounce/Throttle** | ❌ কঠিন | ✅ সহজ |
| **Filter conditions** | ❌ কঠিন | ✅ সহজ |
| **Multiple states** | ⚠️ Complex | ✅ সহজ |
| **Flow operators** | ❌ না | ✅ হ্যাঁ |

---

## 3. Basic Syntax

### 3.1 Minimum Example

```kotlin
@Composable
fun BasicExample() {
    var count by remember { mutableStateOf(0) }
    
    LaunchedEffect(Unit) {
        snapshotFlow { count }  // State → Flow
            .collect { value ->  // Observe করো
                println("Count changed to: $value")
            }
    }
    
    Button(onClick = { count++ }) {
        Text("Count: $count")
    }
}
```

### 3.2 Structure Breakdown

```kotlin
LaunchedEffect(Unit) {           // 1. Coroutine scope
    snapshotFlow { state }       // 2. State observe করো
        .operator1()             // 3. Transform করো (optional)
        .operator2()             // 4. Filter করো (optional)
        .collect { value ->      // 5. Handle করো
            // Side effect here
        }
}
```

### 3.3 Key Points

```kotlin
✅ LaunchedEffect(Unit) - একবার setup
✅ snapshotFlow { } - state track করে
✅ Flow operators - transform/filter করে
✅ .collect { } - final value handle করে
```

---

## 4. Real-World Use Cases

### 4.1 Use Case 1: Search with Debounce

#### Problem
প্রতি keystroke এ API call হচ্ছে → Server overload

#### Solution
```kotlin
@Composable
fun SmartSearchScreen() {
    var searchQuery by remember { mutableStateOf("") }
    var searchResults by remember { mutableStateOf<List<Product>>(emptyList()) }
    var isLoading by remember { mutableStateOf(false) }
    
    LaunchedEffect(Unit) {
        snapshotFlow { searchQuery }
            .debounce(500)                    // User typing থামার জন্য wait
            .filter { it.length >= 3 }        // কমপক্ষে 3 অক্ষর
            .distinctUntilChanged()           // Same query আবার না
            .onEach { isLoading = true }
            .collect { query ->
                println("🔍 Searching: $query")
                searchResults = searchAPI(query)
                isLoading = false
            }
    }
    
    Column {
        TextField(
            value = searchQuery,
            onValueChange = { searchQuery = it },
            label = { Text("Search products") }
        )
        
        if (isLoading) {
            CircularProgressIndicator()
        }
        
        LazyColumn {
            items(searchResults) { product ->
                ProductItem(product)
            }
        }
    }
}
```

**Benefits:**
- ✅ Reduced API calls (500ms debounce)
- ✅ No search until 3 characters
- ✅ Skip duplicate searches
- ✅ Better user experience

---

### 4.2 Use Case 2: Form Validation

#### Problem
প্রতি keystroke এ validation চলছে → UI lag

#### Solution
```kotlin
data class RegistrationForm(
    val name: String = "",
    val email: String = "",
    val password: String = ""
)

@Composable
fun RegistrationScreen() {
    var form by remember { mutableStateOf(RegistrationForm()) }
    var validationError by remember { mutableStateOf<String?>(null) }
    
    // ✅ Debounced validation
    LaunchedEffect(Unit) {
        snapshotFlow { form }
            .debounce(300) // User typing শেষ হওয়ার জন্য wait
            .collect { currentForm ->
                validationError = when {
                    currentForm.name.length < 3 -> 
                        "নাম কমপক্ষে ৩ অক্ষর হতে হবে"
                    !currentForm.email.contains("@") -> 
                        "সঠিক ইমেইল দিন"
                    currentForm.password.length < 8 -> 
                        "পাসওয়ার্ড কমপক্ষে ৮ অক্ষর"
                    else -> null
                }
            }
    }
    
    Column {
        TextField(
            value = form.name,
            onValueChange = { form = form.copy(name = it) },
            label = { Text("নাম") },
            isError = validationError != null
        )
        
        TextField(
            value = form.email,
            onValueChange = { form = form.copy(email = it) },
            label = { Text("ইমেইল") }
        )
        
        TextField(
            value = form.password,
            onValueChange = { form = form.copy(password = it) },
            label = { Text("পাসওয়ার্ড") },
            visualTransformation = PasswordVisualTransformation()
        )
        
        validationError?.let { error ->
            Text(error, color = Color.Red)
        }
        
        Button(
            onClick = { /* Submit */ },
            enabled = validationError == null
        ) {
            Text("Register")
        }
    }
}
```

**Console Output:**
```
User typing: "H" → (wait...)
User typing: "Ha" → (wait...)
(300ms after typing stopped)
❌ Validation: নাম কমপক্ষে ৩ অক্ষর হতে হবে

User continues: "Hasi" → (wait...)
(300ms)
✅ Validation passed!
```

---

### 4.3 Use Case 3: Analytics Tracking

#### Problem
প্রতি scroll/action এ analytics call → Server spam

#### Solution
```kotlin
@Composable
fun ProductScreen(productId: String) {
    var scrollPosition by remember { mutableStateOf(0) }
    val listState = rememberLazyListState()
    
    // ✅ Throttled scroll tracking
    LaunchedEffect(Unit) {
        snapshotFlow { listState.firstVisibleItemIndex }
            .sample(1000) // প্রতি 1 second এ একবার
            .collect { position ->
                println("📊 Analytics: User scrolled to position $position")
                logAnalytics("scroll_position", position)
            }
    }
    
    // ✅ View duration tracking
    LaunchedEffect(Unit) {
        val startTime = System.currentTimeMillis()
        
        snapshotFlow { System.currentTimeMillis() }
            .sample(5000) // প্রতি 5 seconds
            .collect { currentTime ->
                val duration = currentTime - startTime
                println("⏱️ User viewed for ${duration/1000}s")
                logAnalytics("view_duration", duration)
            }
    }
    
    LazyColumn(state = listState) {
        // Product items
    }
}
```

**Benefits:**
- ✅ Throttled analytics (না প্রতি event)
- ✅ Periodic tracking (every 5s)
- ✅ Server-friendly

---

### 4.4 Use Case 4: Auto-Save Feature

#### Problem
প্রতি change এ save হচ্ছে → Too many DB writes

#### Solution
```kotlin
@Composable
fun NoteEditor() {
    var noteContent by remember { mutableStateOf("") }
    var lastSaved by remember { mutableStateOf<Long?>(null) }
    var isSaving by remember { mutableStateOf(false) }
    
    // ✅ Auto-save with debounce
    LaunchedEffect(Unit) {
        snapshotFlow { noteContent }
            .debounce(2000) // 2 seconds পর save
            .filter { it.isNotBlank() }
            .collect { content ->
                isSaving = true
                println("💾 Auto-saving note...")
                
                try {
                    saveNoteToDatabase(content)
                    lastSaved = System.currentTimeMillis()
                    println("✅ Saved successfully!")
                } catch (e: Exception) {
                    println("❌ Save failed: ${e.message}")
                } finally {
                    isSaving = false
                }
            }
    }
    
    Column {
        TextField(
            value = noteContent,
            onValueChange = { noteContent = it },
            label = { Text("Write your note...") },
            modifier = Modifier.fillMaxWidth().height(300.dp)
        )
        
        Row {
            if (isSaving) {
                CircularProgressIndicator(modifier = Modifier.size(16.dp))
                Spacer(Modifier.width(8.dp))
                Text("Saving...")
            } else {
                lastSaved?.let { time ->
                    Text("Last saved: ${formatTime(time)}")
                }
            }
        }
    }
}
```

**Console Output:**
```
User typing: "Hello" → (wait...)
User typing: "Hello World" → (wait...)
(2 seconds after typing stopped)
💾 Auto-saving note...
✅ Saved successfully!
```

---

### 4.5 Use Case 5: Music Player Progress Tracking

#### Problem
গান play হচ্ছে, progress save করতে হবে + auto-next চালাতে হবে

#### Solution
```kotlin
@Composable
fun MusicPlayer(songId: String) {
    var isPlaying by remember { mutableStateOf(false) }
    var progress by remember { mutableStateOf(0) }
    var songDuration by remember { mutableStateOf(180) } // 3 minutes
    
    // ✅ Auto-save progress
    LaunchedEffect(songId) {
        snapshotFlow { progress }
            .filter { isPlaying } // শুধু playing state এ
            .sample(5000) // প্রতি 5 seconds
            .collect { currentProgress ->
                println("💾 Saving progress: $currentProgress/${songDuration}s")
                saveProgressToDatabase(songId, currentProgress)
            }
    }
    
    // ✅ Auto-next when song finishes
    LaunchedEffect(songId) {
        snapshotFlow { progress }
            .filter { it >= songDuration }
            .collect {
                println("🎵 Song finished! Playing next...")
                isPlaying = false
                playNextSong()
            }
    }
    
    // ✅ Update progress while playing
    LaunchedEffect(isPlaying) {
        while (isPlaying && progress < songDuration) {
            delay(1000)
            progress++
        }
    }
    
    Column {
        Text("Progress: ${progress}s / ${songDuration}s")
        
        LinearProgressIndicator(
            progress = progress.toFloat() / songDuration,
            modifier = Modifier.fillMaxWidth()
        )
        
        Row {
            Button(onClick = { isPlaying = !isPlaying }) {
                Text(if (isPlaying) "⏸️ Pause" else "▶️ Play")
            }
            
            Button(onClick = { progress = 0 }) {
                Text("⏮️ Restart")
            }
        }
    }
}
```

**Console Output:**
```
▶️ Play clicked
(5 seconds)
💾 Saving progress: 5/180s
(5 seconds)
💾 Saving progress: 10/180s
...
(Song reaches end)
🎵 Song finished! Playing next...
```

---

### 4.6 Use Case 6: Shopping Cart Auto-Sync

#### Problem
Cart items change করলে server এ sync করতে হবে, কিন্তু প্রতি change এ না

#### Solution
```kotlin
data class CartItem(val id: String, val name: String, val quantity: Int)

@Composable
fun ShoppingCartScreen() {
    var cartItems by remember { mutableStateOf<List<CartItem>>(emptyList()) }
    var syncStatus by remember { mutableStateOf("Synced") }
    
    // ✅ Auto-sync with debounce
    LaunchedEffect(Unit) {
        snapshotFlow { cartItems }
            .debounce(2000) // 2 seconds পর sync
            .collect { items ->
                syncStatus = "Syncing..."
                println("🔄 Syncing cart (${items.size} items)...")
                
                try {
                    syncCartToServer(items)
                    syncStatus = "Synced ✅"
                    println("✅ Cart synced successfully!")
                } catch (e: Exception) {
                    syncStatus = "Sync failed ❌"
                    println("❌ Sync error: ${e.message}")
                }
            }
    }
    
    // ✅ Empty cart warning
    LaunchedEffect(Unit) {
        snapshotFlow { cartItems.isEmpty() }
            .distinctUntilChanged()
            .collect { isEmpty ->
                if (isEmpty) {
                    println("⚠️ Cart is empty!")
                }
            }
    }
    
    Column {
        Text("Cart Status: $syncStatus")
        
        LazyColumn {
            items(cartItems) { item ->
                CartItemRow(
                    item = item,
                    onQuantityChange = { newQuantity ->
                        cartItems = cartItems.map {
                            if (it.id == item.id) it.copy(quantity = newQuantity)
                            else it
                        }
                    },
                    onRemove = {
                        cartItems = cartItems.filter { it.id != item.id }
                    }
                )
            }
        }
        
        Button(onClick = {
            cartItems = cartItems + CartItem(
                id = UUID.randomUUID().toString(),
                name = "New Item",
                quantity = 1
            )
        }) {
            Text("Add Item")
        }
    }
}
```

**Console Output:**
```
User adds item: [Item1]
(wait 2s...)
🔄 Syncing cart (1 items)...
✅ Cart synced successfully!

User adds item: [Item1, Item2]
User adds item: [Item1, Item2, Item3]
(wait 2s...)
🔄 Syncing cart (3 items)...
✅ Cart synced successfully!

User removes all items: []
⚠️ Cart is empty!
```

---

## 5. Flow Operators with snapshotFlow

### 5.1 Available Operators

```kotlin
LaunchedEffect(Unit) {
    snapshotFlow { state }
        .debounce(300)              // Delay করে emit
        .filter { it > 10 }         // Condition match করলে emit
        .distinctUntilChanged()     // Same value skip
        .map { it * 2 }            // Transform করে
        .mapLatest { api(it) }     // Latest value priority
        .sample(1000)              // Fixed interval এ emit
        .take(5)                   // প্রথম 5টা
        .drop(3)                   // প্রথম 3টা skip
        .onEach { log(it) }        // Side effect প্রতিটা value এ
        .catch { e ->              // Error handling
            emit(defaultValue)
        }
        .collect { result ->
            // Handle করো
        }
}
```

### 5.2 Operator Examples

#### debounce
```kotlin
// User typing শেষ হওয়ার জন্য wait
snapshotFlow { searchText }
    .debounce(500) // 500ms delay
    .collect { query -> searchAPI(query) }
```

#### filter
```kotlin
// শুধু condition match করলে emit
snapshotFlow { stock }
    .filter { it < 10 } // Low stock alert
    .collect { showAlert("Low stock: $it") }
```

#### distinctUntilChanged
```kotlin
// Same consecutive values skip
snapshotFlow { userName }
    .distinctUntilChanged()
    .collect { name -> updateProfile(name) }
```

#### map
```kotlin
// Transform করে
snapshotFlow { celsius }
    .map { it * 9/5 + 32 } // Celsius to Fahrenheit
    .collect { fahrenheit -> display(fahrenheit) }
```

#### sample
```kotlin
// Fixed interval এ emit
snapshotFlow { scrollPosition }
    .sample(1000) // প্রতি 1 second
    .collect { pos -> logAnalytics(pos) }
```

#### combine (Multiple states)
```kotlin
LaunchedEffect(Unit) {
    combine(
        snapshotFlow { firstName },
        snapshotFlow { lastName }
    ) { first, last ->
        "$first $last"
    }.collect { fullName ->
        updateDisplay(fullName)
    }
}
```

---

## 6. LaunchedEffect vs snapshotFlow

### 6.1 Comparison Table

| Feature | LaunchedEffect | snapshotFlow |
|---------|---------------|--------------|
| **Purpose** | Side effects with keys | State observation |
| **Restart on key change** | ✅ Yes | ❌ No (continuous) |
| **Flow operators** | ❌ Limited | ✅ Full support |
| **Debounce/Throttle** | ❌ Manual | ✅ Built-in |
| **Multiple state tracking** | ⚠️ Complex | ✅ Easy |
| **Use case** | Simple effects | Complex state observation |

### 6.2 When to Use What

#### ✅ Use LaunchedEffect when:
```kotlin
// Simple one-time effect
LaunchedEffect(userId) {
    val user = fetchUser(userId)
    updateUI(user)
}

// Effect that should restart on key change
LaunchedEffect(productId) {
    loadProductDetails(productId)
}
```

#### ✅ Use snapshotFlow when:
```kotlin
// Need debounce/throttle
LaunchedEffect(Unit) {
    snapshotFlow { searchQuery }
        .debounce(500)
        .collect { search(it) }
}

// Complex state observation
LaunchedEffect(Unit) {
    snapshotFlow { cartItems }
        .filter { it.size > 5 }
        .collect { showBulkDiscount() }
}

// Multiple Flow operators needed
LaunchedEffect(Unit) {
    snapshotFlow { form }
        .debounce(300)
        .distinctUntilChanged()
        .collect { validate(it) }
}
```

---

## 7. Best Practices

### 7.1 Do's ✅

#### 1. Use `LaunchedEffect(Unit)` for Setup
```kotlin
// ✅ Good - একবার setup
LaunchedEffect(Unit) {
    snapshotFlow { state }
        .collect { /* handle */ }
}
```

#### 2. Use Appropriate Operators
```kotlin
// ✅ Good - debounce for user input
snapshotFlow { searchText }
    .debounce(500)
    .collect { search(it) }

// ✅ Good - sample for analytics
snapshotFlow { scrollPos }
    .sample(1000)
    .collect { logScroll(it) }
```

#### 3. Handle Errors
```kotlin
// ✅ Good - error handling
snapshotFlow { query }
    .debounce(300)
    .catch { e ->
        println("Error: ${e.message}")
        emit(emptyList())
    }
    .collect { results ->
        updateUI(results)
    }
```

#### 4. Use distinctUntilChanged for Expensive Operations
```kotlin
// ✅ Good - skip duplicate values
snapshotFlow { userData }
    .distinctUntilChanged()
    .collect { saveToServer(it) }
```

### 7.2 Don'ts ❌

#### 1. Don't Use for Simple Cases
```kotlin
// ❌ Bad - overkill for simple effect
LaunchedEffect(Unit) {
    snapshotFlow { count }
        .collect { println(it) }
}

// ✅ Better - use LaunchedEffect directly
LaunchedEffect(count) {
    println(count)
}
```

#### 2. Don't Forget Debounce for Frequent Changes
```kotlin
// ❌ Bad - too many calls
snapshotFlow { searchText }
    .collect { searchAPI(it) } // Called on every keystroke!

// ✅ Good - debounced
snapshotFlow { searchText }
    .debounce(500)
    .collect { searchAPI(it) }
```

#### 3. Don't Use Multiple LaunchedEffect with Same State
```kotlin
// ❌ Bad - duplicate observation
LaunchedEffect(Unit) {
    snapshotFlow { state }.collect { logIt(it) }
}
LaunchedEffect(Unit) {
    snapshotFlow { state }.collect { saveIt(it) }
}

// ✅ Good - single observation
LaunchedEffect(Unit) {
    snapshotFlow { state }
        .onEach { logIt(it) }
        .collect { saveIt(it) }
}
```

---

## 8. Complete Examples

### 8.1 Complete Search Feature

```kotlin
data class Product(val id: String, val name: String, val price: Double)

@Composable
fun CompleteSearchScreen(
    viewModel: SearchViewModel = viewModel()
) {
    var searchQuery by remember { mutableStateOf("") }
    var searchResults by remember { mutableStateOf<List<Product>>(emptyList()) }
    var isLoading by remember { mutableStateOf(false) }
    var error by remember { mutableStateOf<String?>(null) }
    var searchCount by remember { mutableStateOf(0) }
    
    // Main search flow
    LaunchedEffect(Unit) {
        snapshotFlow { searchQuery }
            .debounce(500)                    // Wait for typing to stop
            .filter { it.length >= 3 }        // Min 3 characters
            .distinctUntilChanged()           // Skip duplicate queries
            .onEach { 
                isLoading = true
                error = null
                searchCount++
                println("🔍 Search #$searchCount: $it")
            }
            .mapLatest { query ->             // Cancel previous search
                try {
                    searchAPI(query)
                } catch (e: Exception) {
                    println("❌ Error: ${e.message}")
                    emptyList()
                }
            }
            .catch { e ->                     // Handle errors
                error = e.message
                emit(emptyList())
            }
            .collect { results ->
                searchResults = results
                isLoading = false
                println("✅ Found ${results.size} results")
            }
    }
    
    // Analytics tracking
    LaunchedEffect(Unit) {
        snapshotFlow { searchQuery }
            .filter { it.isNotEmpty() }
            .debounce(2000)                   // After user finished
            .collect { query ->
                println("📊 Analytics: User searched '$query'")
                logSearchAnalytics(query)
            }
    }
    
    Column(modifier = Modifier.padding(16.dp)) {
        TextField(
            value = searchQuery,
            onValueChange = { searchQuery = it },
            label = { Text("Search products") },
            modifier = Modifier.fillMaxWidth(),
            singleLine = true
        )
        
        Spacer(modifier = Modifier.height(8.dp))
        
        when {
            isLoading -> {
                CircularProgressIndicator()
            }
            error != null -> {
                Text("Error: $error", color = Color.Red)
            }
            searchQuery.length < 3 && searchQuery.isNotEmpty() -> {
                Text("Type at least 3 characters", color = Color.Gray)
            }
            searchResults.isEmpty() && searchQuery.length >= 3 -> {
                Text("No results found", color = Color.Gray)
            }
            else -> {
                Text("Total searches: $searchCount")
                Spacer(modifier = Modifier.height(8.dp))
                
                LazyColumn {
                    items(searchResults) { product ->
                        ProductItem(product)
                    }
                }
            }
        }
    }
}

@Composable
fun ProductItem(product: Product) {
    Row(
        modifier = Modifier
            .fillMaxWidth()
            .padding(vertical = 8.dp)
    ) {
        Column {
            Text(product.name, fontWeight = FontWeight.Bold)
            Text("৳${product.price}", color = Color.Gray)
        }
    }
}
```

---

### 8.2 Complete Form with Validation

```kotlin
data class UserForm(
    val name: String = "",
    val email: String = "",
    val phone: String = "",
    val age: String = ""
)

@Composable
fun CompleteFormScreen() {
    var form by remember { mutableStateOf(UserForm()) }
    var errors by remember { mutableStateOf<Map<String, String>>(emptyMap()) }
    var isValidating by remember { mutableStateOf(false) }
    var formIsValid by remember { mutableStateOf(false) }
    
    // Validation flow
    LaunchedEffect(Unit) {
        snapshotFlow { form }
            .debounce(300)                    // Wait for typing
            .onEach { isValidating = true }
            .collect { currentForm ->
                val newErrors = mutableMapOf<String, String>()
                
                // Name validation
                if (currentForm.name.length < 3) {
                    newErrors["name"] = "নাম কমপক্ষে ৩ অক্ষর হতে হবে"
                }
                
                // Email validation
                if (!currentForm.email.contains("@")) {
                    newErrors["email"] = "সঠিক ইমেইল দিন"
                }
                
                // Phone validation
                if (currentForm.phone.length != 11) {
                    newErrors["phone"] = "ফোন নাম্বার ১১ সংখ্যার হতে হবে"
                }
                
                // Age validation
                val ageInt = currentForm.age.toIntOrNull()
                if (ageInt == null || ageInt < 18) {
                    newErrors["age"] = "বয়স কমপক্ষে ১৮ হতে হবে"
                }
                
                errors = newErrors
                formIsValid = newErrors.isEmpty()
                isValidating = false
                
                println("📝 Validation: ${if (formIsValid) "✅ Valid" else "❌ ${newErrors.size} errors"}")
            }
    }
    
    Column(modifier = Modifier.padding(16.dp)) {
        Text("Registration Form", fontSize = 24.sp, fontWeight = FontWeight.Bold)
        Spacer(modifier = Modifier.height(16.dp))
        
        TextField(
            value = form.name,
            onValueChange = { form = form.copy(name = it) },
            label = { Text("নাম") },
            isError = errors.containsKey("name"),
            modifier = Modifier.fillMaxWidth()
        )
        errors["name"]?.let { Text(it, color = Color.Red, fontSize = 12.sp) }
        
        Spacer(modifier = Modifier.height(8.dp))
        
        TextField(
            value = form.email,
            onValueChange = { form = form.copy(email = it) },
            label = { Text("ইমেইল") },
            isError = errors.containsKey("email"),
            modifier = Modifier.fillMaxWidth()
        )
        errors["email"]?.let { Text(it, color = Color.Red, fontSize = 12.sp) }
        
        Spacer(modifier = Modifier.height(8.dp))
        
        TextField(
            value = form.phone,
            onValueChange = { form = form.copy(phone = it) },
            label = { Text("ফোন") },
            isError = errors.containsKey("phone"),
            modifier = Modifier.fillMaxWidth()
        )
        errors["phone"]?.let { Text(it, color = Color.Red, fontSize = 12.sp) }
        
        Spacer(modifier = Modifier.height(8.dp))
        
        TextField(
            value = form.age,
            onValueChange = { form = form.copy(age = it) },
            label = { Text("বয়স") },
            isError = errors.containsKey("age"),
            modifier = Modifier.fillMaxWidth()
        )
        errors["age"]?.let { Text(it, color = Color.Red, fontSize = 12.sp) }
        
        Spacer(modifier = Modifier.height(16.dp))
        
        Row(verticalAlignment = Alignment.CenterVertically) {
            Button(
                onClick = { /* Submit */ },
                enabled = formIsValid && !isValidating,
                modifier = Modifier.weight(1f)
            ) {
                Text("Submit")
            }
            
            if (isValidating) {
                Spacer(modifier = Modifier.width(8.dp))
                CircularProgressIndicator(modifier = Modifier.size(20.dp))
            }
        }
        
        if (formIsValid) {
            Text("✅ Form is valid!", color = Color.Green)
        }
    }
}
```

---

### 8.3 Complete Music Player with Tracking

```kotlin
data class Song(val id: String, val title: String, val duration: Int)

@Composable
fun CompleteMusicPlayer(songId: String) {
    var currentSong by remember { mutableStateOf<Song?>(null) }
    var isPlaying by remember { mutableStateOf(false) }
    var progress by remember { mutableStateOf(0) }
    var playCount by remember { mutableStateOf(0) }
    
    // Load song
    LaunchedEffect(songId) {
        currentSong = loadSong(songId)
        progress = loadSavedProgress(songId)
    }
    
    // Progress tracking & auto-save
    LaunchedEffect(songId) {
        snapshotFlow { progress }
            .filter { isPlaying }             // Only when playing
            .sample(5000)                     // Every 5 seconds
            .collect { currentProgress ->
                println("💾 Saving progress: $currentProgress/${currentSong?.duration}s")
                saveProgress(songId, currentProgress)
            }
    }
    
    // Song completion detection
    LaunchedEffect(songId) {
        snapshotFlow { progress }
            .filter { currentSong != null && it >= currentSong!!.duration }
            .collect {
                println("🎵 Song completed!")
                isPlaying = false
                playCount++
                logSongCompletion(songId)
                
                // Auto play next
                delay(1000)
                playNextSong()
            }
    }
    
    // Play count milestone tracking
    LaunchedEffect(Unit) {
        snapshotFlow { playCount }
            .filter { it > 0 && it % 10 == 0 } // Every 10 plays
            .collect { count ->
                println("🏆 Milestone! $count plays")
                unlockAchievement("played_${count}_songs")
                showCelebration()
            }
    }
    
    // Progress updater
    LaunchedEffect(isPlaying, currentSong) {
        while (isPlaying && currentSong != null && progress < currentSong.duration) {
            delay(1000)
            progress++
        }
    }
    
    Column(modifier = Modifier.padding(16.dp)) {
        currentSong?.let { song ->
            Text(song.title, fontSize = 20.sp, fontWeight = FontWeight.Bold)
            Spacer(modifier = Modifier.height(8.dp))
            
            Text("${progress}s / ${song.duration}s")
            
            LinearProgressIndicator(
                progress = progress.toFloat() / song.duration,
                modifier = Modifier.fillMaxWidth()
            )
            
            Spacer(modifier = Modifier.height(16.dp))
            
            Row(horizontalArrangement = Arrangement.SpaceEvenly) {
                Button(onClick = { progress = maxOf(0, progress - 10) }) {
                    Text("⏪ -10s")
                }
                
                Button(onClick = { isPlaying = !isPlaying }) {
                    Text(if (isPlaying) "⏸️ Pause" else "▶️ Play")
                }
                
                Button(onClick = { progress = minOf(song.duration, progress + 10) }) {
                    Text("⏩ +10s")
                }
            }
            
            Spacer(modifier = Modifier.height(16.dp))
            
            Text("Total plays: $playCount", color = Color.Gray)
        }
    }
}
```

---

## 9. Summary & Quick Reference

### 9.1 Core Concept
```kotlin
snapshotFlow = Compose State → Kotlin Flow converter

Purpose: State changes observe করে side effects চালানো
```

### 9.2 Basic Template
```kotlin
@Composable
fun MyScreen() {
    var state by remember { mutableStateOf(initialValue) }
    
    LaunchedEffect(Unit) {
        snapshotFlow { state }
            .operator1()  // Transform
            .operator2()  // Filter
            .collect { value ->
                // Handle side effect
            }
    }
}
```

### 9.3 Common Patterns

| Pattern | Code |
|---------|------|
| **Debounced Search** | `.debounce(500).filter { it.length >= 3 }` |
| **Throttled Analytics** | `.sample(1000)` |
| **Validation** | `.debounce(300).collect { validate(it) }` |
| **Auto-save** | `.debounce(2000).collect { save(it) }` |
| **Milestone tracking** | `.filter { it % 10 == 0 }` |
| **Completion detection** | `.filter { it >= threshold }` |

### 9.4 কখন ব্যবহার করবেন

✅ **Use snapshotFlow when:**
- State changes observe করতে হবে
- Debounce/throttle দরকার
- Flow operators দরকার
- Multiple states combine করতে হবে
- Complex side effects

❌ **Don't use when:**
- Simple LaunchedEffect যথেষ্ট
- UI update-ই শুধু দরকার
- One-time action

### 9.5 Performance Tips

```kotlin
✅ Use debounce for user input
✅ Use sample for periodic updates
✅ Use filter to reduce emissions
✅ Use distinctUntilChanged to skip duplicates
✅ Use mapLatest to cancel old operations
```

---

## 10. Additional Resources

### 10.1 Official Documentation
- [Compose Side Effects](https://developer.android.com/jetpack/compose/side-effects)
- [Kotlin Flow](https://kotlinlang.org/docs/flow.html)

### 10.2 Common Mistakes to Avoid

#### Mistake 1: No Debounce on Frequent Changes
```kotlin
// ❌ Bad
snapshotFlow { searchText }.collect { search(it) }

// ✅ Good
snapshotFlow { searchText }.debounce(500).collect { search(it) }
```

#### Mistake 2: Multiple Observations
```kotlin
// ❌ Bad
LaunchedEffect(Unit) {
    snapshotFlow { state }.collect { log(it) }
}
LaunchedEffect(Unit) {
    snapshotFlow { state }.collect { save(it) }
}

// ✅ Good
LaunchedEffect(Unit) {
    snapshotFlow { state }
        .onEach { log(it) }
        .collect { save(it) }
}
```

#### Mistake 3: No Error Handling
```kotlin
// ❌ Bad
snapshotFlow { query }
    .collect { searchAPI(it) } // Crashes on error

// ✅ Good
snapshotFlow { query }
    .catch { e -> emit(emptyList()) }
    .collect { results -> display(results) }
```

---

## শেষ কথা

**snapshotFlow** হলো Compose এর একটি powerful tool যা State observation এবং complex side effects handle করার জন্য ideal। 

**মনে রাখুন:**
- ✅ Simple cases এ LaunchedEffect ব্যবহার করুন
- ✅ Complex state observation এ snapshotFlow ব্যবহার করুন
- ✅ সবসময় debounce/throttle ব্যবহার করুন user input এ
- ✅ Error handling করতে ভুলবেন না

Happy Coding! 🚀

---

**Tutorial শেষ।** কোনো প্রশ্ন বা confusion থাকলে practice করুন এবং experiment করুন! 💪
