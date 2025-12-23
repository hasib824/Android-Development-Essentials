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

### ⚠️ গুরুত্বপূর্ণ Point: Calculated Value Same হলে Variable Update হয় না!

এটা খুবই important একটা concept যা বুঝতে হবে!

**যখন derivedStateOf এর calculation চলে এবং result same পায়, তখন এটা variable কেই update করে না! এইজন্য recomposition হয় না।**

#### কীভাবে কাজ করে:

```kotlin
val category by remember {
    derivedStateOf {
        println("🔴 Calculating category")
        when {
            count < 10 -> "Low"
            count < 50 -> "Medium"
            else -> "High"
        }
    }
}
```

**Internal Process:**

```
Step 1: count change (5 → 6)
Step 2: derivedStateOf dependency detect করলো
Step 3: Calculation lambda চললো
Step 4: Result পেলো: "Low"
Step 5: Previous value check করলো: "Low"
Step 6: Compare করলো: "Low" == "Low"? → Yes!
Step 7: ❌ category variable update করলো না!
Step 8: category না change হলে Text এর dependency change হয়নি
Step 9: তাই Text recompose হয় না! 🎯
```

**আরেকবার:**

```
Step 1: count change (10)
Step 2: derivedStateOf dependency detect করলো
Step 3: Calculation lambda চললো
Step 4: Result পেলো: "Medium"
Step 5: Previous value check করলো: "Low"
Step 6: Compare করলো: "Medium" == "Low"? → No!
Step 7: ✅ category variable update করলো → "Medium"
Step 8: category change হলো → Text এর dependency change
Step 9: তাই Text recompose হয় ✅
```

#### Example: Conditional Calculation

```kotlin
@Composable
fun ConditionalExample() {
    var count by remember { mutableStateOf(0) }
    
    val category by remember {
        derivedStateOf {
            println("🔴 Calculating category")
            when {
                count < 10 -> "Low"
                count < 50 -> "Medium"
                else -> "High"
            }
        }
    }
    
    Column {
        Text("Category: $category")
        println("✅ Text composed")
        
        Button(onClick = { count++ }) {
            Text("Count: $count")
        }
    }
}
```

**Button click করলে:**
```
count = 0 → 
🔴 Calculating category
category variable = "Low"
✅ Text composed

count = 1 → 
🔵 Function executed
🔴 Calculating category (চলেছে)
Result: "Low"
Previous: "Low"
"Low" == "Low"? Yes!
❌ category variable update হয়নি (still "Low")
❌ Text dependency change হয়নি
(✅ Text composed হলো না!) 🎯

count = 2 → 
🔵 Function executed
🔴 Calculating category (চলেছে)
Result: "Low"
Previous: "Low"
❌ category variable update হয়নি
(✅ Text composed হলো না!) 🎯

...

count = 10 → 
🔵 Function executed
🔴 Calculating category (চলেছে)
Result: "Medium"
Previous: "Low"
"Medium" == "Low"? No!
✅ category variable update হলো → "Medium"
✅ Text dependency change হলো
✅ Text composed 🎯

count = 11 → 
🔵 Function executed
🔴 Calculating category (চলেছে)
Result: "Medium"
Previous: "Medium"
❌ category variable update হয়নি
(✅ Text composed হলো না!) 🎯
```

**দেখুন:**
- count 0→1→2→...→9 (10 বার change!)
- Calculation চলেছে 10 বার ✅
- কিন্তু `category` variable এর value সবসময় "Low" ই থাকলো
- `category` update না হওয়ায় Text recompose হয়েছে শুধু 1 বার! (প্রথম load এ) 🎯
- count 10 এ গেলে category "Medium" এ update হলো
- তখন Text recompose হলো ✅

#### Another Example: Boolean Condition

```kotlin
@Composable
fun BooleanExample() {
    var score by remember { mutableStateOf(0) }
    
    val isPassed by remember {
        derivedStateOf {
            println("🔴 Checking pass status")
            score >= 40  // Pass mark is 40
        }
    }
    
    Column {
        Text(if (isPassed) "✅ Passed" else "❌ Failed")
        println("✅ Text composed")
        
        Button(onClick = { score += 5 }) {
            Text("Score: $score")
        }
    }
}
```

**Button clicks:**
```
score = 0 → 
🔴 Checking pass status
isPassed = false
✅ Text composed ("❌ Failed")

score = 5 → 
🔵 Function executed
🔴 Checking pass status (চলেছে)
Result: false
Previous: false
false == false? Yes!
❌ isPassed update হয়নি
(✅ Text composed না!) 🎯

score = 10 → 
🔵 Function executed
🔴 Checking pass status (চলেছে)
❌ isPassed update হয়নি (false)
(✅ Text composed না!) 🎯

score = 35 → 
🔵 Function executed
🔴 Checking pass status (চলেছে)
❌ isPassed update হয়নি (false)
(✅ Text composed না!) 🎯

score = 40 → 
🔵 Function executed
🔴 Checking pass status (চলেছে)
Result: true
Previous: false
true == false? No!
✅ isPassed update হলো → true
✅ Text composed ("✅ Passed") 🎯

score = 45 → 
🔵 Function executed
🔴 Checking pass status (চলেছে)
❌ isPassed update হয়নি (true)
(✅ Text composed না!) 🎯

score = 50 → 
🔵 Function executed
🔴 Checking pass status (চলেছে)
❌ isPassed update হয়নি (true)
(✅ Text composed না!) 🎯
```

**Performance Impact:**
```
Without derivedStateOf:
- 10 score changes → 10 recompositions

With derivedStateOf:
- 10 score changes → 2 recompositions (শুধু value transition এ)
  - false (first time)
  - false → true (at score 40)
- 5x reduction! 🚀
```

**মূল শিক্ষা:**
```
derivedStateOf smart কারণ:

1. Dependency track করে (calculation trigger এর জন্য)
2. Calculation চলে (dependency change হলে)
3. Result compare করে (== দিয়ে)
4. Same হলে → Variable update করে না! 🎯
5. Variable update না হলে → Dependent component recompose হয় না! 🎯

Process:
count change → Calculation → Result same → 
Variable update না → Dependency same → Recompose না! 🎯

এইজন্য expensive operations এ huge benefit! 🎯
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
│  3. Variable Update (NEW!)              │
│     Without derivedStateOf:             │
│     ✅ Calculation চললেই update         │
│                                         │
│     With derivedStateOf:                │
│     ✅ Value comparison (==):           │
│        - Same → Variable update না! 🎯 │
│        - Changed → Variable update     │
└─────────────────────────────────────────┘
           ↓
┌─────────────────────────────────────────┐
│  4. Component Recomposition             │
│     Without derivedStateOf:             │
│     ✅ Variable update হলেই recompose   │
│                                         │
│     With derivedStateOf:                │
│     ✅ Variable update check:           │
│        - না হলে → Recompose skip! 🎯  │
│        - হলে → Recompose              │
└─────────────────────────────────────────┘
```

---

### তাহলে derivedStateOf এর আসল benefit কী?

**derivedStateOf তিনটা optimization করে:**

1. **Calculation Skip** (Dependency same হলে) 🎯
   - CPU save
   - Expensive operations faster
   
2. **Variable Update Skip** (Calculated value same হলে) 🎯🎯
   - এটাই মূল trick!
   - Variable না update হলে dependency change হয় না
   
3. **Component Recomposition Skip** (Variable update না হলে) 🎯
   - UI smooth
   - Battery save

**Performance gain:**
- Function execution: ❌ কমানো যায় না (Compose behavior)
- Calculation: ✅ Skip হয় (Dependency same হলে) - Major gain!
- Variable Update: ✅ Skip হয় (Value same হলে) - This is the key! 🎯
- Component recomposition: ✅ Skip হয় (Variable update না হলে) - Major gain!

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

### ⚠️ Important Note: Value Equality and Variable Update

**derivedStateOf uses structural equality (==) to compare values. যদি calculated value same হয়, তাহলে variable-ই update করে না!**

```kotlin
// Example 1: Primitive types
val category by derivedStateOf {
    if (count < 10) "Low" else "High"
}
// count: 5→6→7 → Calculation চলে 3 বার
// কিন্তু category variable update হয় না (always "Low")
// তাই recompose হয় না! 🎯

// Example 2: Boolean
val isValid by derivedStateOf {
    name.length > 3 && email.contains("@")
}
// name: "ab"→"abc" → Calculation চলে 2 বার
// কিন্তু isValid variable update হয় না (always false)
// তাই recompose হয় না! 🎯

// Example 3: Numbers
val percentage by derivedStateOf {
    (score * 100) / total
}
// score: 40, total: 100 → 40%
// score: 50, total: 125 → 40% (same!)
// percentage variable update হয় না
// তাই recompose হয় না! 🎯
```

**Internal Process:**
```
Step 1: Dependency change detect
Step 2: Run calculation lambda
Step 3: Get new result
Step 4: Compare with previous value (using ==)
Step 5: If same → ❌ DON'T update variable
        If different → ✅ Update variable
Step 6: Variable না update হলে → Dependent components recompose না
```

**মূল Point:**
```
Dependency change করলেই variable update হয় না!
Calculated value change হলেই variable update হয়!
Variable update হলেই recomposition হয়!

3-step protection:
1. Dependency same → Calculation skip
2. Calculation চললেও value same → Variable update skip 🎯
3. Variable update না হলে → Recomposition skip

এইজন্য derivedStateOf এত powerful! 🚀
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

### Real Performance Example: Category System

```kotlin
@Composable
fun ProductList() {
    var price by remember { mutableStateOf(0) }
    
    // Without derivedStateOf
    val categorySimple = when {
        price < 100 -> "Budget"
        price < 500 -> "Mid-range"
        else -> "Premium"
    }
    
    // With derivedStateOf
    val categoryDerived by remember {
        derivedStateOf {
            when {
                price < 100 -> "Budget"
                price < 500 -> "Mid-range"
                else -> "Premium"
            }
        }
    }
}
```

**Performance Analysis:**

```
User slides price: 0→10→20→30→40→50→60→70→80→90 (10 changes)

Without derivedStateOf:
- Calculations: 10
- Recompositions: 10
- categorySimple: "Budget" all 10 times (same value!)
❌ 10 unnecessary recompositions!

With derivedStateOf:
- Calculations: 10
- Recompositions: 1 (only first time)
- categoryDerived: "Budget" (detected same value, skipped 9 recompositions!)
✅ 9 recompositions saved!

User continues: 90→100→200→300→400 (4 more changes)

Without derivedStateOf:
- Recompositions: 4 more = Total 14

With derivedStateOf:
- 100: "Mid-range" (changed! recompose) ✅
- 200→300→400: "Mid-range" (same! skip 3) 🎯
- Recompositions: 1 more = Total 2

Final Score:
Without: 14 recompositions
With: 2 recompositions
Savings: 85% reduction! 🚀
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
✅ Variable update prevent করে (Calculated value same হলে) 🎯
✅ Component recomposition prevent করে (Variable update না হলে) 🎯

⚠️ Critical Understanding:
Dependency change → Calculation চলে → Value same → 
→ Variable update হয় না! → Recomposition হয় না! 🎯

Example 1:
Switch toggle → Function execute → derivedStateOf check করে
             → Dependency same? → Calculation skip 🎯
             → Variable update না → Recomposition না 🎯
             
Example 2:
count 5→6 → Function execute → derivedStateOf check করে
         → Dependency changed → Calculation চলে ✅
         → Result: "Low"
         → Previous: "Low" 
         → Value same → category variable update হয় না! 🎯
         → Variable update না → Recomposition না! 🎯

count 10 → Function execute → derivedStateOf check করে
        → Dependency changed → Calculation চলে ✅
        → Result: "Medium"
        → Previous: "Low"
        → Value changed → category variable update হয় ✅
        → Variable update হলো → Recomposition হয় ✅
```

### 3. কেন দরকার?
```
✅ Unnecessary Calculation বন্ধ করে (Dependency same হলে)
✅ Unnecessary Variable Update বন্ধ করে (Calculated value same হলে) 🎯
✅ Unnecessary Component recomposition বন্ধ করে (Variable update না হলে)
✅ Expensive operations optimize করে (CPU save)
✅ Expensive UI updates reduce করে
✅ UI smooth রাখে
✅ Battery efficient

মূল Concept:
Dependency change → Calculation চলে → Value same → 
Variable update না → Recomposition না! 🎯
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
3. Calculation runs (if dependency changed)
4. Result compared with previous value using == 🎯
5. Same result → Variable UPDATE SKIPPED! 🎯🎯
6. Variable না update হলে → Dependent components don't recompose ✅
7. Changed result → Variable updated → Recomposition happens ✅

Critical Understanding:
⚠️ Dependency change ≠ Variable update
⚠️ Variable update ≠ যদি calculated value same
✅ Variable update = Recomposition trigger

Flow:
count: 5→6→7 (3 dependency changes)
Calculation: "Low"→"Low"→"Low" (3 calculations)
category variable: "Low" (NO UPDATE! stays same) 🎯
Result: 0 recompositions! 🚀

count: 10 (1 dependency change)
Calculation: "Medium" (1 calculation)
category variable: "Low" → "Medium" (UPDATED!) ✅
Result: 1 recomposition! ✅

Purpose:
- State থেকে calculated value তৈরি করে
- Dependency same → Calculation skip 🎯
- Value same → Variable update skip 🎯
- Variable update না হলে → Recomposition skip 🎯
- Triple protection = Maximum performance!

Usage:
val result by remember {
    derivedStateOf {
        calculate(state1, state2)
    }
}

Remember:
✅ Prevents Calculation (when dependency same)
✅ Prevents Variable Update (when calculated value same) 🎯
✅ Prevents Recomposition (when variable not updated)
❌ Cannot prevent Function execution
🎯 Uses == for value comparison (structural equality)

Key Point:
derivedStateOf = তিনটা layer protection:
                1. Calculation skip (dependency same)
                2. Variable update skip (value same) 🎯
                3. Recomposition skip (variable unchanged)
                
Variable update না হওয়াই মূল trick! 🎯
এইজন্য dependency change হলেও recompose হয় না! 🚀
```
✅ Use for state-dependent calculations
✅ Use for expensive operations
❌ Don't overuse for simple things

Key Point:
derivedStateOf = Function চলবে, Calculation চলবে
                কিন্তু UI update শুধু দরকার হলেই! 🎯
```

**Happy Coding! 🚀**
