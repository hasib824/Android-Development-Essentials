# Kotlin Scope Functions - সহজ বাংলা টিউটোরিয়াল

## সূচিপত্র
1. [Scope Function কী?](#scope-function-কী)
2. [let](#1-let)
3. [run](#2-run)
4. [with](#3-with)
5. [apply](#4-apply)
6. [also](#5-also)
7. [তুলনামূলক সারণী](#তুলনামূলক-সারণী)

---

## Scope Function কী?

Scope Function হলো এমন কিছু function যা একটি অবজেক্টের উপর **temporary scope** তৈরি করে। এই scope এর ভিতরে আপনি অবজেক্টটিকে সহজে access করতে পারেন।

### মোট ৫টি Scope Function:
1. **let** - Null checking ও transformation এর জন্য
2. **run** - Object configuration এবং result calculate করার জন্য
3. **with** - একই object এ multiple operation করার জন্য
4. **apply** - Object configure করার জন্য
5. **also** - Additional operations করার জন্য

---

## ১. let

### কী করে?
- Object কে `it` হিসেবে পায়
- Lambda এর **শেষ expression** return করে
- **Null safety** এর জন্য সবচেয়ে বেশি ব্যবহৃত হয়

### কখন ব্যবহার করবেন?
✅ Null checking করতে  
✅ Local scope variable তৈরি করতে  
✅ Function call চেইন করতে

### সহজ উদাহরণ:

```kotlin
// ❌ Without let
val name: String? = getUserName()
if (name != null) {
    println("নাম: ${name.uppercase()}")
    saveToDatabase(name)
}

// ✅ With let
getUserName()?.let { name ->
    println("নাম: ${name.uppercase()}")
    saveToDatabase(name)
}
```

### বাস্তব উদাহরণ:

```kotlin
// Example 1: Null safe operation
val user: User? = getUser()
user?.let {
    println("ইউজার নাম: ${it.name}")
    println("ইউজার ইমেইল: ${it.email}")
}

// Example 2: Transform এবং return
val length = text?.let {
    it.trim().length
} ?: 0

// Example 3: Function chaining
val result = getPhoneNumber()
    ?.let { it.trim() }
    ?.let { it.replace("-", "") }
    ?.let { "+88$it" }

// Example 4: File reading
File("data.txt").takeIf { it.exists() }?.let { file ->
    val content = file.readText()
    println(content)
}

// Example 5: API call
viewModelScope.launch {
    userRepository.getUser(userId)?.let { user ->
        _uiState.value = UiState.Success(user)
    }
}
```

---

## ২. run

### কী করে?
- Object কে `this` হিসেবে পায়
- Lambda এর **শেষ expression** return করে
- Object এবং extension function উভয়ভাবে কাজ করে

### কখন ব্যবহার করবেন?
✅ Object initialization এবং result compute করতে  
✅ Multiple properties access করতে  
✅ Expression এর result দরকার হলে

### সহজ উদাহরণ:

```kotlin
// Extension function হিসেবে
val userName = user.run {
    "$firstName $lastName"
}

// Non-extension হিসেবে (code block execute)
val result = run {
    val a = 10
    val b = 20
    a + b  // 30 return হবে
}
```

### বাস্তব উদাহরণ:

```kotlin
// Example 1: Object properties access
val greeting = user.run {
    "হ্যালো $name, আপনার ইমেইল $email"
}

// Example 2: TextView configuration
textView.run {
    text = "কোটলিন শিখুন"
    textSize = 20f
    setTextColor(Color.BLUE)
}

// Example 3: Calculate করে result return
val discount = product.run {
    val tax = price * 0.15
    val discountAmount = price * 0.10
    price + tax - discountAmount
}

// Example 4: Nullable object এ operation
val info = user?.run {
    "নাম: $name\nবয়স: $age\nঠিকানা: $address"
} ?: "ইউজার পাওয়া যায়নি"

// Example 5: SharedPreferences
val isLoggedIn = run {
    val prefs = context.getSharedPreferences("app", Context.MODE_PRIVATE)
    prefs.getBoolean("logged_in", false)
}
```

---

## ৩. with

### কী করে?
- Object কে `this` হিসেবে পায়
- Lambda এর **শেষ expression** return করে
- **Extension function নয়** - regular function

### কখন ব্যবহার করবেন?
✅ একই object এ multiple operations করতে  
✅ "এই object নিয়ে এটা করো" - এই ধরনের logic এ  
✅ Non-null object এর জন্য

### সহজ উদাহরণ:

```kotlin
val person = Person()

// ❌ Without with
person.name = "রহিম"
person.age = 25
person.city = "ঢাকা"

// ✅ With with
with(person) {
    name = "রহিম"
    age = 25
    city = "ঢাকা"
}
```

### বাস্তব উদাহরণ:

```kotlin
// Example 1: TextView setup
with(textView) {
    text = "স্বাগতম"
    textSize = 18f
    setTextColor(Color.BLACK)
    gravity = Gravity.CENTER
}

// Example 2: Canvas drawing
with(canvas) {
    drawColor(Color.WHITE)
    drawCircle(50f, 50f, 25f, paint)
    drawText("Hello", 100f, 100f, textPaint)
}

// Example 3: StringBuilder
val message = with(StringBuilder()) {
    append("নাম: ")
    append(user.name)
    append("\n")
    append("ইমেইল: ")
    append(user.email)
    toString()  // Return হবে
}

// Example 4: Calculate multiple values
val result = with(calculator) {
    add(10)
    multiply(5)
    subtract(20)
    getResult()
}

// Example 5: RecyclerView setup
with(recyclerView) {
    layoutManager = LinearLayoutManager(context)
    adapter = myAdapter
    setHasFixedSize(true)
    addItemDecoration(DividerItemDecoration(context, DividerItemDecoration.VERTICAL))
}
```

---

## ৪. apply

### কী করে?
- Object কে `this` হিসেবে পায়
- **Object নিজেই** return করে
- Object configuration এর জন্য সবচেয়ে জনপ্রিয়

### কখন ব্যবহার করবেন?
✅ Object create করে configure করতে  
✅ Builder pattern এর মতো কাজ করতে  
✅ Object return করতে হলে

### সহজ উদাহরণ:

```kotlin
// Object তৈরি এবং configure
val person = Person().apply {
    name = "করিম"
    age = 30
    email = "karim@example.com"
}
```

### বাস্তব উদাহরণ:

```kotlin
// Example 1: Intent তৈরি
val intent = Intent(this, MainActivity::class.java).apply {
    putExtra("USER_ID", userId)
    putExtra("USER_NAME", userName)
    flags = Intent.FLAG_ACTIVITY_NEW_TASK
}
startActivity(intent)

// Example 2: TextView তৈরি এবং setup
val textView = TextView(context).apply {
    text = "কোটলিন"
    textSize = 20f
    setTextColor(Color.RED)
    layoutParams = ViewGroup.LayoutParams(MATCH_PARENT, WRAP_CONTENT)
}
container.addView(textView)

// Example 3: AlertDialog তৈরি
val dialog = AlertDialog.Builder(context).apply {
    setTitle("নিশ্চিত করুন")
    setMessage("আপনি কি মুছে ফেলতে চান?")
    setPositiveButton("হ্যাঁ") { _, _ -> deleteItem() }
    setNegativeButton("না", null)
}.create()
dialog.show()

// Example 4: File তৈরি
val file = File(path).apply {
    createNewFile()
    writeText("এটা একটা টেস্ট ফাইল")
}

// Example 5: User object setup
val newUser = User().apply {
    id = generateId()
    name = nameEditText.text.toString()
    email = emailEditText.text.toString()
    createdAt = System.currentTimeMillis()
}
saveUser(newUser)
```

---

## ৫. also

### কী করে?
- Object কে `it` হিসেবে পায়
- **Object নিজেই** return করে
- Additional operations বা side-effects এর জন্য

### কখন ব্যবহার করবেন?
✅ Object এর উপর additional operations করতে  
✅ Logging, validation করতে  
✅ Object modify না করে extra কাজ করতে

### সহজ উদাহরণ:

```kotlin
val numbers = mutableListOf(1, 2, 3)
    .also { println("লিস্ট তৈরি হয়েছে: $it") }
    .also { it.add(4) }
    .also { println("আপডেটেড লিস্ট: $it") }
```

### বাস্তব উদাহরণ:

```kotlin
// Example 1: Logging
val user = getUser()
    .also { println("ইউজার পাওয়া গেছে: ${it.name}") }
    .also { logUserAccess(it.id) }

// Example 2: Validation করে save
fun saveUser(user: User) {
    user.also {
        require(it.name.isNotEmpty()) { "নাম খালি থাকতে পারবে না" }
        require(it.email.contains("@")) { "ইমেইল সঠিক নয়" }
    }.also {
        database.save(it)
        println("ইউজার সেভ হয়েছে: ${it.id}")
    }
}

// Example 3: File operations
File("config.json")
    .also { println("ফাইল পাথ: ${it.absolutePath}") }
    .also { if (!it.exists()) it.createNewFile() }
    .writeText(jsonContent)

// Example 4: List operations
val filteredUsers = users
    .filter { it.age > 18 }
    .also { println("ফিল্টারড ইউজার: ${it.size}") }
    .sortedBy { it.name }
    .also { println("সর্ট করা হয়েছে") }

// Example 5: API response logging
suspend fun fetchData(): List<Item> {
    return apiService.getItems()
        .also { println("API থেকে ${it.size}টি আইটেম পাওয়া গেছে") }
        .also { cacheItems(it) }
}
```

---

## কোনটা কখন ব্যবহার করবেন? (সহজ গাইড)

### 🎯 **Null Safety দরকার?**
→ **let** ব্যবহার করুন
```kotlin
user?.let { 
    println(it.name) 
}
```

### 🎯 **Object Configure করতে হবে?**
→ **apply** ব্যবহার করুন
```kotlin
val person = Person().apply {
    name = "রহিম"
    age = 25
}
```

### 🎯 **একই Object এ অনেক কাজ করতে হবে?**
→ **with** ব্যবহার করুন
```kotlin
with(textView) {
    text = "Hello"
    textSize = 20f
}
```

### 🎯 **Result Calculate করতে হবে?**
→ **run** ব্যবহার করুন
```kotlin
val total = cart.run {
    price + tax - discount
}
```

### 🎯 **Logging/Debugging করতে হবে?**
→ **also** ব্যবহার করুন
```kotlin
data.also { 
    println("Data: $it") 
}
```

---

## বাস্তব Scenario Examples

### Scenario 1: User Registration
```kotlin
fun registerUser(name: String, email: String, password: String): User {
    return User().apply {
        this.id = generateId()
        this.name = name
        this.email = email
        this.passwordHash = hashPassword(password)
        this.createdAt = System.currentTimeMillis()
    }.also {
        println("নতুন ইউজার: ${it.name}")
        database.save(it)
    }.also {
        sendWelcomeEmail(it.email)
    }
}
```

### Scenario 2: Image Loading
```kotlin
fun loadImage(url: String, imageView: ImageView) {
    url.let { imageUrl ->
        Glide.with(context)
            .load(imageUrl)
            .apply(RequestOptions().centerCrop())
            .into(imageView)
    }.also {
        println("ছবি লোড হচ্ছে: $url")
    }
}
```

### Scenario 3: SharedPreferences
```kotlin
fun saveUserData(user: User) {
    with(sharedPreferences.edit()) {
        putString("user_name", user.name)
        putString("user_email", user.email)
        putInt("user_age", user.age)
        apply()
    }
}

fun getUserData(): User? {
    return sharedPreferences.run {
        val name = getString("user_name", null)
        val email = getString("user_email", null)
        val age = getInt("user_age", 0)
        
        if (name != null && email != null) {
            User(name, email, age)
        } else {
            null
        }
    }
}
```

### Scenario 4: RecyclerView ViewHolder
```kotlin
class UserViewHolder(view: View) : RecyclerView.ViewHolder(view) {
    private val nameText: TextView = view.findViewById(R.id.nameText)
    private val emailText: TextView = view.findViewById(R.id.emailText)
    
    fun bind(user: User) {
        with(user) {
            nameText.text = name
            emailText.text = email
        }
        
        itemView.apply {
            setOnClickListener { onUserClick(user) }
            setOnLongClickListener { 
                onUserLongClick(user)
                true
            }
        }
    }
}
```

### Scenario 5: Network Call
```kotlin
suspend fun fetchUserData(userId: String): Result<User> {
    return try {
        apiService.getUser(userId)
            .also { println("API Response: $it") }
            .let { response ->
                if (response.isSuccessful) {
                    response.body()?.let { user ->
                        Result.Success(user)
                    } ?: Result.Error("ইউজার পাওয়া যায়নি")
                } else {
                    Result.Error("Error: ${response.code()}")
                }
            }
    } catch (e: Exception) {
        Result.Error(e.message ?: "Unknown error")
    }
}
```

---

## তুলনামূলক সারণী

### 📊 সংক্ষিপ্ত তুলনা

| Function | Context | Return | Use Case |
|----------|---------|--------|----------|
| **let** | `it` | Lambda result | Null safety, transform |
| **run** | `this` | Lambda result | Calculate result |
| **with** | `this` | Lambda result | Group operations |
| **apply** | `this` | Object itself | Configure object |
| **also** | `it` | Object itself | Side effects, logging |

### 📋 বিস্তারিত তুলনা

| বৈশিষ্ট্য | let | run | with | apply | also |
|---------|-----|-----|------|-------|------|
| **Context Object** | it | this | this | this | it |
| **Return Value** | Lambda result | Lambda result | Lambda result | Context object | Context object |
| **Extension Function** | ✅ হ্যাঁ | ✅ হ্যাঁ | ❌ না | ✅ হ্যাঁ | ✅ হ্যাঁ |
| **Null Safety** | ✅ চমৎকার | ✅ ভালো | ❌ না | ⚠️ সাবধান | ⚠️ সাবধান |
| **Best For** | Null check | Result compute | Group function calls | Object config | Side effects |

### 🎨 Context Object তুলনা

```kotlin
val user = User("রহিম", 25)

// "it" ব্যবহার করে (let, also)
user.let { 
    println(it.name)  // it দিয়ে access
}

user.also { 
    println(it.name)  // it দিয়ে access
}

// "this" ব্যবহার করে (run, apply, with)
user.run {
    println(name)  // সরাসরি access (this.name)
}

user.apply {
    name = "করিম"  // সরাসরি assign (this.name = ...)
}

with(user) {
    println(name)  // সরাসরি access
}
```

### 🔄 Return Value তুলনা

```kotlin
val numbers = mutableListOf(1, 2, 3)

// Lambda result return করে
val doubled = numbers.let { 
    it.map { num -> num * 2 }  
}  // Result: [2, 4, 6]

val sum = numbers.run { 
    sum() 
}  // Result: 6

val result = with(numbers) { 
    size 
}  // Result: 3

// Object নিজেই return করে
val list1 = numbers.apply { 
    add(4) 
}  // Result: [1, 2, 3, 4]

val list2 = numbers.also { 
    println(it) 
}  // Result: [1, 2, 3, 4]
```

### 💡 সিদ্ধান্ত নেওয়ার চার্ট

```
আপনার Object কি Nullable?
├─ হ্যাঁ → let ব্যবহার করুন
└─ না
   └─ আপনি কী return চান?
      ├─ Object নিজে
      │  ├─ Configuration করতে হবে? → apply
      │  └─ Logging/Debugging? → also
      │
      └─ Lambda এর result
         ├─ Extension function লাগবে? → run
         └─ Extension function লাগবে না? → with
```

---

## চূড়ান্ত টিপস

### ✅ **করুন:**
```kotlin
// Chain করুন যখন প্রয়োজন
user?.let { it.validate() }
    ?.also { println("Validated") }
    ?.apply { status = "active" }

// Null safety তে let ব্যবহার করুন
name?.let { println(it) }

// Object config এ apply ব্যবহার করুন
TextView(context).apply { 
    text = "Hello" 
}
```

### ❌ **করবেন না:**
```kotlin
// অপ্রয়োজনে scope function ব্যবহার
val name = user.let { it.name }  // ❌
val name = user.name             // ✅

// Nested scope function (পড়া কঠিন)
user.let { u ->
    u.address.let { a ->
        a.city.let { c ->
            println(c)  // ❌ জটিল
        }
    }
}

// সহজ করুন
user.address.city.let { println(it) }  // ✅
```

---

## সারাংশ

### 📝 মনে রাখার সূত্র:

1. **let** = **L**ambda result + **I**t + Nul**L** safety
2. **run** = **R**esult compute + **U**se this + Ru**N**
3. **with** = **W**ork **I**n group + **Th**is context
4. **apply** = **App**ly configuration + Return object
5. **also** = **Al**so do this + Return **o**bject

### 🎯 সবচেয়ে বেশি ব্যবহৃত:

1. **let** - Null checking (90% সময়)
2. **apply** - Object configuration (80% সময়)
3. **also** - Logging/Debugging (60% সময়)
4. **with** - Multiple operations (40% সময়)
5. **run** - Complex calculations (30% সময়)

### 💪 Practice করুন:

প্রতিটি function এর জন্য নিজে ৫টি করে উদাহরণ লিখুন। তাহলে পুরোপুরি clear হয়ে যাবে!

---

**Happy Coding! 🚀**

*Scope functions আপনার Kotlin code কে আরো readable এবং concise করে তুলবে।*
