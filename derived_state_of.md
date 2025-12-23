# derivedStateOf - সংক্ষিপ্ত Tutorial 🎯

## সূচিপত্র

1. [derivedStateOf কী?](#1-derivedstateof-কী)
2. [কেন derivedStateOf দরকার?](#2-কেন-derivedstateof-দরকার)
3. [মূল সমস্যা ও সমাধান](#3-মূল-সমস্যা-ও-সমাধান)
4. [Real-World Use Cases](#4-real-world-use-cases)
5. [remember vs derivedStateOf](#5-remember-vs-derivedstateof)
6. [Quick Reference](#6-quick-reference)

---

## 1. derivedStateOf কী?

### সংজ্ঞা
**derivedStateOf** = একটি Compose API যা state থেকে **derived/calculated value** তৈরি করে এবং **smart caching** করে।

### Real Life Analogy 🏪

**দোকানদারের হিসাব:**

```
আপনার ঝুড়িতে:
- আম: 3টা (50 টাকা করে)
- কলা: 5টা (10 টাকা করে)

মোট = (3 × 50) + (5 × 10) = 200 টাকা
```

**Without derivedStateOf (সাধারণ দোকানদার):**
```
আপনি ঝুড়ি নাড়ালেন → দোকানদার আবার হিসাব করলো → 200 টাকা
আপনি আম দেখলেন → দোকানদার আবার হিসাব করলো → 200 টাকা
আপনি দাঁড়ালেন → দোকানদার আবার হিসাব করলো → 200 টাকা

❌ প্রতিবার হিসাব, কিন্তু result same!
```

**With derivedStateOf (স্মার্ট দোকানদার):**
```
প্রথমবার হিসাব → 200 টাকা → মনে রাখলো

আপনি ঝুড়ি নাড়ালেন → "পণ্য পরিবর্তন হয়নি?" → হিসাব করলো না
আপনি ১টা কলা বেশি নিলেন → "পণ্য পরিবর্তন হয়েছে!" → নতুন হিসাব → 210 টাকা

✅ শুধু actual পরিবর্তন হলেই হিসাব!
```

---

## 2. কেন derivedStateOf দরকার?

### মূল কারণ: Unnecessary Calculation এবং Component Recomposition - দুইটাই বন্ধ করা!

**🎯 derivedStateOf এর দুইটা Main Power:**

```
1️⃣ Calculation Skip (Dependency same হলে)
   → CPU save
   → Expensive operations faster
   → 1000 items sort না করা!

2️⃣ Component Recomposition Skip (Value same হলে)  
   → UI smooth
   → Battery save
   → Unnecessary redraw না হওয়া!

Result: Double optimization = Major performance boost! 🚀
```

---

**⚠️ গুরুত্বপূর্ণ বুঝতে হবে: Function Execution vs Component Recomposition**

Compose এ যখন কোনো state change হয়, **দুইটি জিনিস** ঘটে:

#### 1️⃣ Function Execution (সবসময় হয়)
```kotlin
@Composable
fun MyScreen() {  // ← এই function টা আবার call হয়
    println("🔵 Function executed")
    var count by remember { mutableStateOf(0) }
    
    val doubled = count * 2  // ← এই calculation আবার চলে
    
    Text("Count: $doubled")
}
```

**State change হলে:**
- ✅ পুরো `MyScreen()` function আবার execute হয়
- ⚠️ `val doubled = count * 2` line আবার চলে (derivedStateOf বন্ধ করতে পারে!)
- ⚠️ **Function execution বন্ধ করতে পারে না, কিন্তু calculation skip করতে পারে!**

#### 2️⃣ Component Recomposition (শুধু dependency change হলে)
```kotlin
@Composable
fun MyScreen() {
    var count by remember { mutableStateOf(0) }
    var name by remember { mutableStateOf("Hasi") }
    
    val doubled = count * 2
    
    Column {
        Text("Count: $doubled")  // ← শুধু doubled change হলে recompose
        Text("Name: $name")       // ← শুধু name change হলে recompose
    }
}
```

**count change হলে:**
- ✅ Function execute → `doubled` recalculate
- ✅ Text("Count: $doubled") recompose করে
- ❌ Text("Name: $name") recompose করে না (dependency নেই)

---

### derivedStateOf আসলে কী করে?

#### ❌ Without derivedStateOf

```kotlin
@Composable
fun Example() {
    var count by remember { mutableStateOf(0) }
    var toggle by remember { mutableStateOf(false) }
    
    println("🔵 Function executed")
    
    // Calculation চলে প্রতিবার
    val doubled = count * 2
    println("🔴 Calculation: $count * 2 = $doubled")
    
    Column {
        Text("Doubled: $doubled")
        println("✅ Text composed")
        
        Switch(toggle, { toggle = it })
    }
}
```

**Switch toggle করলে:**
```
Console Output:
🔵 Function executed          ← Function চললো
🔴 Calculation: 0 * 2 = 0    ← Calculation চললো
✅ Text composed              ← Text recompose হলো

❌ সমস্যা: 
   1. Calculation আবার চলেছে (কিন্তু count same!)
   2. doubled এর value change হয়নি (0 → 0)
   3. Text recompose হয়েছে (unnecessary!)
```

#### ✅ With derivedStateOf

```kotlin
@Composable
fun Example() {
    var count by remember { mutableStateOf(0) }
    var toggle by remember { mutableStateOf(false) }
    
    println("🔵 Function executed")
    
    val doubled by remember {
        derivedStateOf {
            println("🔴 Calculation: $count * 2")
            count * 2
        }
    }
    
    Column {
        Text("Doubled: $doubled")
        println("✅ Text composed")
        
        Switch(toggle, { toggle = it })
    }
}
```

**Switch toggle করলে:**
```
Console Output:
🔵 Function executed                ← Function চললো (এটা সবসময় হবে)
(🔴 Calculation চলেনি!)             ← Calculation skip! ✅
(✅ Text composed হলো না!)          ← Text recompose হয়নি! ✅

✅ Benefit: 
   1. derivedStateOf dependency check করলো (count same?)
   2. count change হয়নি, তাই calculation এর lambda-ই চলেনি!
   3. Cached value ব্যবহার করলো
   4. Value same (0), তাই Text recompose করলো না!
```

**count change করলে:**
```
Console Output:
🔵 Function executed                ← Function চললো
🔴 Calculation: 1 * 2              ← এবার calculation চললো! ✅
✅ Text composed                    ← Text recompose হলো! ✅

✅ Correct: count change হয়েছে, তাই calculation চললো এবং Text update হলো
```

---

### Visual Summary: কী কী হয়?

```
State Change হলে:

┌─────────────────────────────────────────┐
│  1. Function Execution                  │
│     ✅ সবসময় হয়                        │
│     ❌ derivedStateOf বন্ধ করতে পারে না │
└─────────────────────────────────────────┘
           ↓
┌─────────────────────────────────────────┐
│  2. Calculation                         │
│     Without derivedStateOf:             │
│     ✅ প্রতিবার চলে                     │
│                                         │
│     With derivedStateOf:                │
│     ✅ Dependency check করে:            │
│        - Same → Calculation skip! 🎯   │
│        - Changed → Calculation চলে     │
└─────────────────────────────────────────┘
           ↓
┌─────────────────────────────────────────┐
│  3. Component Recomposition             │
│     Without derivedStateOf:             │
│     ✅ Calculation চললে recompose       │
│                                         │
│     With derivedStateOf:                │
│     ✅ Value check করে:                 │
│        - Same → Recompose skip! 🎯     │
│        - Changed → Recompose           │
└─────────────────────────────────────────┘
```

---

### তাহলে derivedStateOf এর আসল benefit কী?

**derivedStateOf দুইটা optimization করে:**

1. **Calculation Skip** (Dependency same হলে) 🎯
   - CPU save
   - Expensive operations faster
   
2. **Component Recomposition Skip** (Value same হলে) 🎯
   - UI smooth
   - Battery save

**Performance gain:**
- Function execution: ❌ কমানো যায় না (Compose behavior)
- Calculation: ✅ Skip হয় (Dependency same হলে) - Major gain!
- Component recomposition: ✅ Skip হয় (Value same হলে) - Major gain!

---


## 3. Real-World Use Cases

### Use Case 1: Todo List - Completed Count

#### ❌ Problem
```kotlin
@Composable
fun TodoList() {
    var todos by remember { mutableStateOf(getTodos()) }
    
    // প্রতিবার todos change → recalculate
    val completedCount = todos.count { it.done }
    
    Column {
        Text("Completed: $completedCount/${todos.size}")
        
        todos.forEach { todo ->
            TodoItem(
                todo = todo,
                onToggle = { 
                    todos = todos.map {
                        if (it.id == todo.id) it.copy(done = !it.done)
                        else it
                    }
                }
            )
        }
    }
}
```

**সমস্যা:**
```
Todo এর text edit করলেন:
→ todos পরিবর্তন
→ completedCount recalculate (কিন্তু value same!)
→ Text recompose (unnecessary!)
```

#### ✅ Solution
```kotlin
@Composable
fun TodoList() {
    var todos by remember { mutableStateOf(getTodos()) }
    
    // Smart calculation - শুধু count change হলে recompose
    val completedCount by remember {
        derivedStateOf {
            todos.count { it.done }
        }
    }
    
    Column {
        Text("Completed: $completedCount/${todos.size}")
        
        todos.forEach { todo ->
            TodoItem(
                todo = todo,
                onToggle = { 
                    todos = todos.map {
                        if (it.id == todo.id) it.copy(done = !it.done)
                        else it
                    }
                }
            )
        }
    }
}
```

**Benefits:**
- ✅ Todo text edit → completedCount same → No recompose
- ✅ Todo checkbox toggle → completedCount change → Recompose
- 🚀 Performance improved!

---

### Use Case 2: Shopping Cart - Total Price

#### ❌ Problem
```kotlin
data class CartItem(val name: String, val price: Double, val quantity: Int)

@Composable
fun ShoppingCart() {
    var items by remember { mutableStateOf(getCartItems()) }
    var showDiscount by remember { mutableStateOf(false) }
    
    // Expensive calculation
    val totalPrice = items.sumOf { it.price * it.quantity }
    val discount = if (totalPrice > 500) totalPrice * 0.1 else 0.0
    val finalPrice = totalPrice - discount
    
    Column {
        Text("Total: ৳$finalPrice")
        
        Switch(showDiscount, { showDiscount = it })
        
        if (showDiscount) {
            Text("Discount: ৳$discount")
        }
        
        items.forEach { item ->
            CartItemRow(item)
        }
    }
}
```

**সমস্যা:**
```
Switch toggle:
→ showDiscount পরিবর্তন
→ totalPrice, discount, finalPrice আবার calculate
→ কিন্তু values same! (150 টাকা)
→ Text recompose (unnecessary!)
```

#### ✅ Solution
```kotlin
@Composable
fun ShoppingCart() {
    var items by remember { mutableStateOf(getCartItems()) }
    var showDiscount by remember { mutableStateOf(false) }
    
    val finalPrice by remember {
        derivedStateOf {
            val total = items.sumOf { it.price * it.quantity }
            val discount = if (total > 500) total * 0.1 else 0.0
            total - discount
        }
    }
    
    Column {
        Text("Total: ৳$finalPrice")
        
        Switch(showDiscount, { showDiscount = it })
        
        if (showDiscount) {
            val discount = if (finalPrice > 500) finalPrice * 0.1 else 0.0
            Text("Discount: ৳$discount")
        }
        
        items.forEach { item ->
            CartItemRow(item)
        }
    }
}
```

**Benefits:**
- ✅ Switch toggle → finalPrice same → No recompose
- ✅ Item quantity change → finalPrice change → Recompose
- 🚀 Expensive calculation optimized!

---

### Use Case 3: Search Filter - Filtered List

#### ❌ Problem
```kotlin
@Composable
fun ContactList() {
    var searchQuery by remember { mutableStateOf("") }
    var contacts by remember { mutableStateOf(loadContacts()) } // 1000+ contacts
    var sortOrder by remember { mutableStateOf("name") }
    
    // Expensive filtering
    val filteredContacts = contacts.filter { 
        it.name.contains(searchQuery, ignoreCase = true)
    }
    
    Column {
        TextField(searchQuery, { searchQuery = it })
        
        Button(onClick = { sortOrder = if (sortOrder == "name") "date" else "name" }) {
            Text("Sort: $sortOrder")
        }
        
        LazyColumn {
            items(filteredContacts) { contact ->
                ContactItem(contact)
            }
        }
    }
}
```

**সমস্যা:**
```
Sort button click:
→ sortOrder পরিবর্তন
→ filteredContacts recalculate (1000+ contacts filter!)
→ কিন্তু result same list!
→ LazyColumn recompose (unnecessary!)
```

#### ✅ Solution
```kotlin
@Composable
fun ContactList() {
    var searchQuery by remember { mutableStateOf("") }
    var contacts by remember { mutableStateOf(loadContacts()) }
    var sortOrder by remember { mutableStateOf("name") }
    
    val filteredContacts by remember {
        derivedStateOf {
            contacts.filter { 
                it.name.contains(searchQuery, ignoreCase = true)
            }
        }
    }
    
    Column {
        TextField(searchQuery, { searchQuery = it })
        
        Button(onClick = { sortOrder = if (sortOrder == "name") "date" else "name" }) {
            Text("Sort: $sortOrder")
        }
        
        LazyColumn {
            items(filteredContacts) { contact ->
                ContactItem(contact)
            }
        }
    }
}
```

**Benefits:**
- ✅ Sort button click → filteredContacts same → No refiltering
- ✅ Search query change → filteredContacts change → Filter and recompose
- 🚀 1000+ contacts না filter করা!

---

### Use Case 4: Form - Is Valid State

```kotlin
@Composable
fun RegistrationForm() {
    var name by remember { mutableStateOf("") }
    var email by remember { mutableStateOf("") }
    var password by remember { mutableStateOf("") }
    var agreeTerms by remember { mutableStateOf(false) }
    
    // Derived validation state
    val isFormValid by remember {
        derivedStateOf {
            name.length >= 3 &&
            email.contains("@") &&
            password.length >= 8 &&
            agreeTerms
        }
    }
    
    Column {
        TextField(name, { name = it }, label = { Text("Name") })
        TextField(email, { email = it }, label = { Text("Email") })
        TextField(password, { password = it }, label = { Text("Password") })
        
        Row {
            Checkbox(agreeTerms, { agreeTerms = it })
            Text("I agree to terms")
        }
        
        // Button শুধু isFormValid change হলে recompose
        Button(
            onClick = { submitForm() },
            enabled = isFormValid
        ) {
            Text("Submit")
        }
    }
}
```

**Benefits:**
- Button enable/disable efficiently updated
- শুধু validation result change হলেই button recompose

---

## 5. remember vs derivedStateOf

### Comparison Table

| Feature | `remember { }` | `derivedStateOf { }` |
|---------|----------------|---------------------|
| **Calculate কখন** | শুধু প্রথমবার | Dependency change এ |
| **Dependency track** | ❌ না | ✅ হ্যাঁ |
| **Result same হলে** | N/A | Recompose skip করে |
| **Use case** | Static value | Dynamic calculation |

### Example Comparison

```kotlin
@Composable
fun ComparisonExample() {
    var count by remember { mutableStateOf(0) }
    
    // ❌ remember - শুধু প্রথমবার, count change হলেও update হবে না
    val doubled1 = remember { count * 2 }
    
    // ✅ derivedStateOf - count change হলে recalculate, result same হলে skip
    val doubled2 by remember { derivedStateOf { count * 2 } }
    
    Column {
        Text("Doubled1: $doubled1") // সবসময় 0 (wrong!)
        Text("Doubled2: $doubled2") // Correct value
        
        Button(onClick = { count++ }) {
            Text("Count: $count")
        }
    }
}
```

---

## 6. Quick Reference

### কখন ব্যবহার করবেন

#### ✅ Use derivedStateOf when:

```kotlin
1. State depend করে এমন calculation
val total by derivedStateOf { items.sumOf { it.price } }

2. Expensive calculation
val sorted by derivedStateOf { list.sortedBy { it.name } }

3. Multiple states থেকে একটা result
val isValid by derivedStateOf { name.isNotEmpty() && email.contains("@") }

4. Filter/search operations
val filtered by derivedStateOf { items.filter { it.active } }
```

#### ❌ DON'T use when:

```kotlin
1. Simple property access
// ❌ Unnecessary
val name by derivedStateOf { user.name }
// ✅ Better
val name = user.name

2. No state dependency
// ❌ Use remember instead
val id by derivedStateOf { UUID.randomUUID() }
// ✅ Better
val id = remember { UUID.randomUUID() }

3. Side effects দরকার
// ❌ Wrong - use LaunchedEffect
val result by derivedStateOf { 
    apiCall() // Side effect!
}
```

### Basic Template

```kotlin
@Composable
fun MyScreen() {
    var state1 by remember { mutableStateOf(value1) }
    var state2 by remember { mutableStateOf(value2) }
    
    // Derived state - automatically updates when state1 or state2 change
    val derivedValue by remember {
        derivedStateOf {
            // Calculate based on state1, state2
            expensiveCalculation(state1, state2)
        }
    }
    
    // Use derivedValue in UI
    Text("Result: $derivedValue")
}
```

### Performance Impact

```kotlin
Without derivedStateOf:
- Function recompose → Recalculate → Component recompose
- Total: 100 recompositions (wasteful)

With derivedStateOf:
- Function recompose → Check if result changed → Only recompose if needed
- Total: 10 recompositions (optimized)

Performance gain: 10x faster! 🚀
```

---

## মূল শিক্ষা (Key Takeaways)

### 1. What is derivedStateOf?
```
State থেকে calculated value তৈরি করে
Result same হলে Component recomposition skip করে
Performance optimization tool
```

### 2. কী prevent করে?
```
❌ Function execution prevent করে না (এটা সবসময় হবে)
✅ Calculation prevent করে (Dependency same হলে) 🎯
✅ Component recomposition prevent করে (Value same হলে) 🎯

Example:
Switch toggle → Function execute → derivedStateOf check করে
             → Dependency same? → Calculation skip 🎯
             → Value same? → Component recompose করে না 🎯
             
count change → Function execute → derivedStateOf check করে
            → Dependency changed? → Calculation চলে ✅
            → Value changed? → Component recompose করে ✅
```

### 3. কেন দরকার?
```
✅ Unnecessary Calculation বন্ধ করে (Dependency same হলে)
✅ Unnecessary Component recomposition বন্ধ করে (Value same হলে)
✅ Expensive operations optimize করে (CPU save)
✅ Expensive UI updates reduce করে
✅ UI smooth রাখে
✅ Battery efficient
```

### 4. কখন ব্যবহার?
```
✅ State-dependent calculations
✅ List filtering/sorting
✅ Form validation
✅ Total/sum/count calculations
✅ Expensive operations যেখানে result same থাকতে পারে
```

### 5. কখন না?
```
❌ Simple property access
❌ Static values (use remember)
❌ Side effects (use LaunchedEffect)
❌ যেখানে calculation cheap এবং always different result
```

### 6. Performance বুঝি
```
Without derivedStateOf:
Function execute → Calculate প্রতিবার → Component recompose
Total: 100 calculations + 100 recompositions

With derivedStateOf:
Function execute → Dependency check
                → Same? Skip calculation 🎯 + Skip recompose 🎯
                → Changed? Calculate ✅ + Recompose ✅
Total: 10 calculations + 10 recompositions

Performance gain: 
- 90 unnecessary calculations saved (CPU save!)
- 90 unnecessary recompositions saved (UI smooth!)
- 10x faster! 🚀
```

---

## Final Example: Complete Todo App

```kotlin
data class Todo(val id: Int, val text: String, val done: Boolean)

@Composable
fun TodoApp() {
    var todos by remember { mutableStateOf(getTodos()) }
    var filter by remember { mutableStateOf("all") } // all, active, completed
    
    // Derived state 1: Filtered todos
    val filteredTodos by remember {
        derivedStateOf {
            when (filter) {
                "active" -> todos.filter { !it.done }
                "completed" -> todos.filter { it.done }
                else -> todos
            }
        }
    }
    
    // Derived state 2: Stats
    val completedCount by remember {
        derivedStateOf { todos.count { it.done } }
    }
    
    val activeCount by remember {
        derivedStateOf { todos.count { !it.done } }
    }
    
    Column {
        // Stats bar - শুধু counts change হলে recompose
        Row {
            Text("Total: ${todos.size}")
            Text("Active: $activeCount")
            Text("Done: $completedCount")
        }
        
        // Filter buttons
        Row {
            Button(onClick = { filter = "all" }) { Text("All") }
            Button(onClick = { filter = "active" }) { Text("Active") }
            Button(onClick = { filter = "completed" }) { Text("Done") }
        }
        
        // List - শুধু filteredTodos change হলে recompose
        LazyColumn {
            items(filteredTodos) { todo ->
                TodoItem(
                    todo = todo,
                    onToggle = {
                        todos = todos.map {
                            if (it.id == todo.id) it.copy(done = !it.done)
                            else it
                        }
                    }
                )
            }
        }
    }
}
```

**Benefits:**
- ✅ Filter change → শুধু list recompose, stats না
- ✅ Todo toggle → stats update, filtered list smart update
- ✅ Optimized performance
- 🚀 Smooth UI experience

---

## Summary

```kotlin
derivedStateOf = Smart Calculator for Compose

What it does:
1. State change → Function executes (can't prevent)
2. Dependency check → Same? Skip calculation! 🎯
3. Result compared with previous value (if calculated)
4. Same result → Component recomposition SKIPPED ✅
5. Changed result → Component recomposition happens ✅

Purpose:
- State থেকে calculated value তৈরি করে
- Dependency same → Calculation skip 🎯
- Value same → Component recomposition skip 🎯
- Performance boost = CPU + UI save!

Usage:
val result by remember {
    derivedStateOf {
        calculate(state1, state2)
    }
}

Remember:
✅ Prevents Calculation (when dependency same)
✅ Prevents Component recomposition (when value same)
❌ Cannot prevent Function execution

Key Point:
derivedStateOf = দুইটা optimization:
                1. Calculation skip (dependency same)
                2. Recomposition skip (value same)
                🎯 Perfect for expensive operations!
```
✅ Use for state-dependent calculations
✅ Use for expensive operations
❌ Don't overuse for simple things

Key Point:
derivedStateOf = Function চলবে, Calculation চলবে
                কিন্তু UI update শুধু দরকার হলেই! 🎯
```

**Happy Coding! 🚀**
