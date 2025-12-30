# 🎯 Data Binding - সম্পূর্ণ বাংলা Tutorial

একদম scratch থেকে Data Binding শিখি! 🚀

---

## 📖 Table of Contents

1. [Data Binding কি?](#data-binding-কি)
2. [View Binding vs Data Binding](#view-binding-vs-data-binding)
3. [Step by Step Setup](#step-by-step-setup)
4. [Type 1: One-Way Binding (Simple)](#type-1-one-way-binding-simple)
5. [Type 2: Two-Way Binding](#type-2-two-way-binding)
6. [Type 3: Observable Data](#type-3-observable-data)
7. [Type 4: LiveData Binding](#type-4-livedata-binding)
8. [Type 5: Binding Adapters](#type-5-binding-adapters)
9. [Type 6: Event Binding](#type-6-event-binding)
10. [Advanced Features](#advanced-features)
11. [Best Practices](#best-practices)
12. [Summary](#summary)

---

## 🤔 Data Binding কি?

### সহজ ভাষায়:

```
Data Binding = "XML layout এ directly data connect করা"

View Binding:
- শুধু view access করা
- findViewById() এর replacement
- XML → Kotlin (one direction)

Data Binding:
- View + Data দুটোই handle করে
- XML layout এ data ব্যবহার করা
- XML ↔ Kotlin (both directions)
- Observable data support
- Two-way binding support
```

---

## 📊 View Binding vs Data Binding

```
┌─────────────────────────────────────────────────────────────┐
│           View Binding vs Data Binding                      │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ View Binding:                                               │
│ ✅ Simple                                                    │
│ ✅ Fast                                                      │
│ ✅ findViewById() replacement only                          │
│ ✅ Less features                                             │
│ ❌ No XML expressions                                        │
│ ❌ No two-way binding                                        │
│ ❌ No Observable support                                     │
│                                                             │
│ Data Binding:                                               │
│ ✅ Powerful                                                  │
│ ✅ XML expressions                                           │
│ ✅ Two-way binding                                           │
│ ✅ Observable data                                           │
│ ✅ LiveData support                                          │
│ ✅ BindingAdapters                                           │
│ ⚠️ Slower build time                                         │
│ ⚠️ More complex                                              │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### কখন কোনটা ব্যবহার করবে?

```
Use View Binding when:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ Simple view access needed
✅ No data binding required
✅ Fast build time important
✅ Simple projects

Use Data Binding when:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ Need XML expressions
✅ Need two-way binding
✅ Using MVVM architecture
✅ Observable data required
✅ Complex UI logic
```

---

## 📝 Step by Step Setup

### Step 1: Enable Data Binding

```gradle
// app/build.gradle (Module level)

android {
    ...
    
    buildFeatures {
        dataBinding = true  // ✅ Enable Data Binding
    }
}

// Click "Sync Now"
```

### Step 2: What Changed?

```
After enabling Data Binding:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ Can use <layout> tag in XML
✅ Can use <data> tag for variables
✅ Can use @ and @{} expressions
✅ Can use two-way binding @={}
✅ Binding classes auto-generated
✅ All View Binding features included
```

---

## 🎯 Type 1: One-Way Binding (Simple)

### সবচেয়ে basic Data Binding!

### Step 1: Create Data Class

```kotlin
// User.kt
data class User(
    val name: String,
    val email: String,
    val age: Int,
    val isActive: Boolean
)
```

---

### Step 2: Create XML Layout

```xml
<?xml version="1.0" encoding="utf-8"?>
<!-- ✅ Wrap everything in <layout> tag -->
<layout xmlns:android="http://schemas.android.com/apk/res/android">
    
    <!-- ✅ Define variables in <data> tag -->
    <data>
        <variable
            name="user"
            type="com.example.databindingexample.User"/>
    </data>
    
    <!-- ✅ Your actual layout -->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical"
        android:padding="16dp">
        
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="User Profile"
            android:textSize="24sp"
            android:textStyle="bold"
            android:layout_marginBottom="16dp"/>
        
        <!-- ✅ Use @{} to bind data -->
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="@{user.name}"
            android:textSize="18sp"/>
        
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="@{user.email}"
            android:textSize="16sp"/>
        
        <!-- ✅ Convert to String -->
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="@{String.valueOf(user.age)}"
            android:textSize="16sp"/>
        
        <!-- ✅ String concatenation -->
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="@{`Age: ` + String.valueOf(user.age) + ` years`}"
            android:textSize="16sp"/>
        
        <!-- ✅ Conditional (ternary operator) -->
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="@{user.isActive ? `Active User` : `Inactive User`}"
            android:textColor="@{user.isActive ? @color/green : @color/red}"
            android:textSize="16sp"/>
        
    </LinearLayout>
</layout>
```

---

### Step 3: Use in Activity

```kotlin
// MainActivity.kt

class MainActivity : AppCompatActivity() {
    
    // ✅ Declare binding
    private lateinit var binding: ActivityMainBinding
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // ✅ Use DataBindingUtil to inflate
        binding = DataBindingUtil.setContentView(this, R.layout.activity_main)
        
        // ✅ Create user object
        val user = User(
            name = "Hasibuzzaman Chowdhury",
            email = "hasib@example.com",
            age = 25,
            isActive = true
        )
        
        // ✅ Set user to binding
        binding.user = user
        
        // 🎉 That's it! UI automatically updated!
    }
}
```

---

### What Happened?

```
Flow:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

1. Created User data class
   ↓
2. XML layout with <data> and @{} expressions
   ↓
3. Set binding.user = user
   ↓
4. UI automatically shows:
   - Name: "Hasibuzzaman Chowdhury"
   - Email: "hasib@example.com"
   - Age: "Age: 25 years"
   - Status: "Active User" (green color)

Magic:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
❌ No binding.tvName.text = user.name
❌ No binding.tvEmail.text = user.email
❌ No binding.tvAge.text = user.age.toString()

✅ Just: binding.user = user
✅ XML handles everything!
```

---

## 🎯 Type 2: Two-Way Binding

### XML থেকে data change হলে Kotlin এ automatically update!

### Step 1: Create Observable Data Class

```kotlin
// User.kt
import androidx.databinding.BaseObservable
import androidx.databinding.Bindable
import androidx.databinding.library.baseAdapters.BR

data class User(
    private var _name: String = "",
    private var _email: String = "",
    private var _age: Int = 0
) : BaseObservable() {
    
    // ✅ Bindable property with getter/setter
    @get:Bindable
    var name: String
        get() = _name
        set(value) {
            _name = value
            notifyPropertyChanged(BR.name)  // ✅ Notify UI
        }
    
    @get:Bindable
    var email: String
        get() = _email
        set(value) {
            _email = value
            notifyPropertyChanged(BR.email)
        }
    
    @get:Bindable
    var age: Int
        get() = _age
        set(value) {
            _age = value
            notifyPropertyChanged(BR.age)
        }
}
```

---

### Step 2: XML with Two-Way Binding

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">
    
    <data>
        <variable
            name="user"
            type="com.example.databindingexample.User"/>
    </data>
    
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical"
        android:padding="16dp">
        
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Two-Way Binding Example"
            android:textSize="24sp"
            android:textStyle="bold"
            android:layout_marginBottom="24dp"/>
        
        <!-- ✅ Two-way binding with @={} -->
        <EditText
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="Enter name"
            android:text="@={user.name}"
            android:layout_marginBottom="16dp"/>
        
        <EditText
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="Enter email"
            android:text="@={user.email}"
            android:inputType="textEmailAddress"
            android:layout_marginBottom="16dp"/>
        
        <EditText
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="Enter age"
            android:text="@={String.valueOf(user.age)}"
            android:inputType="number"
            android:layout_marginBottom="32dp"/>
        
        <!-- ✅ Display live updates -->
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Live Preview:"
            android:textSize="18sp"
            android:textStyle="bold"
            android:layout_marginBottom="8dp"/>
        
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="@{`Name: ` + user.name}"/>
        
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="@{`Email: ` + user.email}"/>
        
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="@{`Age: ` + String.valueOf(user.age)}"/>
        
    </LinearLayout>
</layout>
```

---

### Step 3: Use in Activity

```kotlin
class MainActivity : AppCompatActivity() {
    
    private lateinit var binding: ActivityMainBinding
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        binding = DataBindingUtil.setContentView(this, R.layout.activity_main)
        
        // ✅ Create observable user
        val user = User(
            name = "Hasibuzzaman",
            email = "hasib@example.com",
            age = 25
        )
        
        binding.user = user
        
        // 🎉 Now when user types in EditText:
        // - user.name automatically updates
        // - TextView automatically shows new value
        // - No code needed!
    }
}
```

---

### What is Two-Way Binding?

```
One-Way Binding (@{}):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Kotlin → XML
user.name changes → TextView updates
But: EditText changes → user.name NOT updated

Two-Way Binding (@={}):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Kotlin ↔ XML
user.name changes → EditText updates
EditText changes → user.name updates
Both directions automatically!

Symbol difference:
@{user.name}  = One-way (read only)
@={user.name} = Two-way (read & write)
```

---

## 🎯 Type 3: Observable Data (More Options)

### Option 1: BaseObservable (Already shown above)

### Option 2: ObservableField

```kotlin
// UserViewModel.kt
import androidx.databinding.ObservableField
import androidx.databinding.ObservableInt
import androidx.databinding.ObservableBoolean

class UserViewModel {
    
    // ✅ Observable fields (easier than BaseObservable)
    val name = ObservableField<String>("Hasibuzzaman")
    val email = ObservableField<String>("hasib@example.com")
    val age = ObservableInt(25)
    val isActive = ObservableBoolean(true)
    
    // Methods to update
    fun updateName(newName: String) {
        name.set(newName)
    }
    
    fun updateAge(newAge: Int) {
        age.set(newAge)
    }
}
```

---

### XML for ObservableField:

```xml
<layout>
    <data>
        <variable
            name="viewModel"
            type="com.example.databindingexample.UserViewModel"/>
    </data>
    
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical"
        android:padding="16dp">
        
        <!-- ✅ Access ObservableField values -->
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="@{viewModel.name}"/>
        
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="@{viewModel.email}"/>
        
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="@{String.valueOf(viewModel.age)}"/>
        
        <!-- ✅ Two-way binding with ObservableField -->
        <EditText
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="@={viewModel.name}"/>
        
    </LinearLayout>
</layout>
```

---

### Activity:

```kotlin
class MainActivity : AppCompatActivity() {
    
    private lateinit var binding: ActivityMainBinding
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        binding = DataBindingUtil.setContentView(this, R.layout.activity_main)
        
        // ✅ Create ViewModel
        val viewModel = UserViewModel()
        binding.viewModel = viewModel
        
        // ✅ Update data programmatically
        viewModel.updateName("New Name")
        // UI automatically updates!
    }
}
```

---

## 🎯 Type 4: LiveData Binding

### Best for MVVM Architecture!

### Step 1: Create ViewModel with LiveData

```kotlin
// UserViewModel.kt
import androidx.lifecycle.ViewModel
import androidx.lifecycle.MutableLiveData
import androidx.lifecycle.LiveData

class UserViewModel : ViewModel() {
    
    // ✅ Private MutableLiveData
    private val _name = MutableLiveData<String>("Hasibuzzaman")
    private val _email = MutableLiveData<String>("hasib@example.com")
    private val _age = MutableLiveData<Int>(25)
    
    // ✅ Public LiveData (read-only)
    val name: LiveData<String> = _name
    val email: LiveData<String> = _email
    val age: LiveData<Int> = _age
    
    // ✅ Methods to update
    fun updateName(newName: String) {
        _name.value = newName
    }
    
    fun updateEmail(newEmail: String) {
        _email.value = newEmail
    }
    
    fun updateAge(newAge: Int) {
        _age.value = newAge
    }
    
    // ✅ Example: Load data from repository
    fun loadUserData() {
        // Simulate network call
        _name.value = "Updated Name"
        _email.value = "updated@email.com"
        _age.value = 30
    }
}
```

---

### Step 2: XML with LiveData

```xml
<layout>
    <data>
        <variable
            name="viewModel"
            type="com.example.databindingexample.UserViewModel"/>
    </data>
    
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical"
        android:padding="16dp">
        
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="LiveData Binding Example"
            android:textSize="24sp"
            android:textStyle="bold"
            android:layout_marginBottom="24dp"/>
        
        <!-- ✅ Bind LiveData directly -->
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="@{viewModel.name}"
            android:textSize="18sp"/>
        
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="@{viewModel.email}"
            android:textSize="16sp"/>
        
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="@{`Age: ` + String.valueOf(viewModel.age)}"
            android:textSize="16sp"
            android:layout_marginBottom="24dp"/>
        
        <Button
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Load Data"
            android:onClick="@{() -> viewModel.loadUserData()}"/>
        
    </LinearLayout>
</layout>
```

---

### Step 3: Activity with LiveData

```kotlin
class MainActivity : AppCompatActivity() {
    
    private lateinit var binding: ActivityMainBinding
    private lateinit var viewModel: UserViewModel
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        binding = DataBindingUtil.setContentView(this, R.layout.activity_main)
        
        // ✅ Get ViewModel
        viewModel = ViewModelProvider(this).get(UserViewModel::class.java)
        
        // ✅ Set ViewModel to binding
        binding.viewModel = viewModel
        
        // ✅ IMPORTANT: Set lifecycle owner for LiveData
        binding.lifecycleOwner = this
        
        // 🎉 Now LiveData automatically updates UI!
        
        // Example: Update data after 3 seconds
        Handler(Looper.getMainLooper()).postDelayed({
            viewModel.updateName("New Name After 3 Seconds")
            // UI automatically updates!
        }, 3000)
    }
}
```

---

### Why LiveData is Best?

```
LiveData Benefits:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ Lifecycle aware (no memory leaks)
✅ Automatic UI updates
✅ No manual observe() in XML
✅ Works with MVVM
✅ Thread-safe
✅ Observer pattern built-in

Must remember:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
binding.lifecycleOwner = this
↑ Without this, LiveData won't update UI!
```

---

## 🎯 Type 5: Binding Adapters

### Custom attributes for views!

### Example 1: Load Image from URL

```kotlin
// BindingAdapters.kt
import android.widget.ImageView
import androidx.databinding.BindingAdapter
import com.bumptech.glide.Glide

object BindingAdapters {
    
    // ✅ Custom attribute: app:imageUrl
    @JvmStatic
    @BindingAdapter("imageUrl")
    fun loadImage(imageView: ImageView, url: String?) {
        if (!url.isNullOrEmpty()) {
            Glide.with(imageView.context)
                .load(url)
                .placeholder(R.drawable.placeholder)
                .error(R.drawable.error)
                .into(imageView)
        }
    }
    
    // ✅ Custom attribute: app:isVisible
    @JvmStatic
    @BindingAdapter("isVisible")
    fun setVisibility(view: View, isVisible: Boolean) {
        view.visibility = if (isVisible) View.VISIBLE else View.GONE
    }
    
    // ✅ Custom attribute: app:formattedDate
    @JvmStatic
    @BindingAdapter("formattedDate")
    fun setFormattedDate(textView: TextView, timestamp: Long) {
        val format = SimpleDateFormat("dd MMM yyyy, hh:mm a", Locale.getDefault())
        textView.text = format.format(Date(timestamp))
    }
}
```

---

### XML using Binding Adapters:

```xml
<layout xmlns:app="http://schemas.android.com/apk/res-auto">
    
    <data>
        <variable
            name="user"
            type="com.example.databindingexample.User"/>
    </data>
    
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical"
        android:padding="16dp">
        
        <!-- ✅ Use custom attribute -->
        <ImageView
            android:layout_width="100dp"
            android:layout_height="100dp"
            app:imageUrl="@{user.profileImageUrl}"/>
        
        <!-- ✅ Conditional visibility -->
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Premium User"
            app:isVisible="@{user.isPremium}"/>
        
        <!-- ✅ Formatted date -->
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            app:formattedDate="@{user.registeredDate}"/>
        
    </LinearLayout>
</layout>
```

---

## 🎯 Type 6: Event Binding (Click Handlers)

### Option 1: Method Reference

```xml
<layout>
    <data>
        <variable
            name="handler"
            type="com.example.databindingexample.ClickHandler"/>
    </data>
    
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical"
        android:padding="16dp">
        
        <!-- ✅ Method reference -->
        <Button
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Click Me"
            android:onClick="@{handler::onButtonClick}"/>
        
    </LinearLayout>
</layout>
```

```kotlin
class ClickHandler(private val context: Context) {
    
    fun onButtonClick(view: View) {
        Toast.makeText(context, "Button clicked!", Toast.LENGTH_SHORT).show()
    }
}

// In Activity:
binding.handler = ClickHandler(this)
```

---

### Option 2: Lambda Expression

```xml
<layout>
    <data>
        <variable
            name="viewModel"
            type="com.example.databindingexample.UserViewModel"/>
    </data>
    
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical"
        android:padding="16dp">
        
        <!-- ✅ Lambda with no parameters -->
        <Button
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Submit"
            android:onClick="@{() -> viewModel.submitData()}"/>
        
        <!-- ✅ Lambda with parameters -->
        <Button
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Delete User"
            android:onClick="@{() -> viewModel.deleteUser(user.id)}"/>
        
    </LinearLayout>
</layout>
```

---

## 🎯 Advanced Features

### Feature 1: Null Safety in XML

```xml
<!-- ✅ Safe call operator -->
<TextView
    android:text="@{user.address?.city ?? `Unknown`}"/>

<!-- ✅ Elvis operator -->
<TextView
    android:text="@{user.name ?? `No Name`}"/>

<!-- ✅ Null check -->
<TextView
    android:text="@{user.name != null ? user.name : `Guest`}"/>
```

---

### Feature 2: Imports in XML

```xml
<data>
    <!-- ✅ Import classes -->
    <import type="android.view.View"/>
    <import type="android.text.TextUtils"/>
    <import type="java.util.List"/>
    
    <variable name="user" type="com.example.User"/>
</data>

<!-- Use imported classes -->
<TextView
    android:visibility="@{user.isActive ? View.VISIBLE : View.GONE}"/>

<TextView
    android:text="@{TextUtils.isEmpty(user.name) ? `No name` : user.name}"/>
```

---

### Feature 3: Collections

```xml
<data>
    <variable
        name="users"
        type="java.util.List&lt;com.example.User&gt;"/>
</data>

<!-- ✅ Access list items -->
<TextView
    android:text="@{users[0].name}"/>

<TextView
    android:text="@{`Total users: ` + String.valueOf(users.size())}"/>
```

---

### Feature 4: Includes

```xml
<!-- main_layout.xml -->
<layout>
    <data>
        <variable name="user" type="com.example.User"/>
    </data>
    
    <LinearLayout...>
        
        <!-- ✅ Include with data passing -->
        <include
            layout="@layout/user_header"
            bind:user="@{user}"/>
        
    </LinearLayout>
</layout>

<!-- user_header.xml -->
<layout>
    <data>
        <variable name="user" type="com.example.User"/>
    </data>
    
    <TextView
        android:text="@{user.name}"/>
</layout>
```

---

## 💡 Best Practices

### ✅ DO:

```kotlin
// ✅ Use Data Binding for MVVM
class MyViewModel : ViewModel() {
    val name = MutableLiveData<String>()
}

// ✅ Set lifecycle owner
binding.lifecycleOwner = this

// ✅ Use BindingAdapters for reusable logic
@BindingAdapter("imageUrl")
fun loadImage(view: ImageView, url: String?) { }

// ✅ Keep expressions simple in XML
android:text="@{user.name}"  // Good
android:text="@{user.name.toUpperCase().substring(0, 5)}"  // Bad

// ✅ Use imports for View constants
<import type="android.view.View"/>
android:visibility="@{View.VISIBLE}"
```

---

### ❌ DON'T:

```kotlin
// ❌ Don't put business logic in XML
android:onClick="@{() -> database.saveUser(user)}"  // Bad!

// ❌ Don't forget lifecycle owner
binding.viewModel = viewModel
// Without binding.lifecycleOwner = this, LiveData won't work!

// ❌ Don't use complex expressions in XML
android:text="@{user.firstName + ` ` + user.middleName + ` ` + user.lastName}"  // Bad
// Instead, create a computed property in ViewModel

// ❌ Don't mix View Binding and Data Binding
// Choose one per project
```

---

## 🎓 Summary

### Data Binding Types:

```
1. One-Way Binding (@{})
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   - Data → UI
   - Simple binding
   - Read only
   - Example: @{user.name}

2. Two-Way Binding (@={})
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   - Data ↔ UI
   - Bidirectional
   - Read & write
   - Example: @={user.name}

3. Observable Data
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   - BaseObservable
   - ObservableField
   - Auto UI updates
   - notifyPropertyChanged()

4. LiveData Binding
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   - Lifecycle aware
   - MVVM pattern
   - Best practice
   - Needs: binding.lifecycleOwner = this

5. Binding Adapters
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   - Custom attributes
   - Reusable logic
   - @BindingAdapter
   - Example: app:imageUrl

6. Event Binding
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   - Click handlers
   - Lambda expressions
   - Method references
   - Example: android:onClick="@{() -> viewModel.submit()}"
```

---

### Quick Reference:

```
Setup:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
buildFeatures { dataBinding = true }

XML Structure:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
<layout>
    <data>
        <variable name="user" type="User"/>
    </data>
    <LinearLayout>
        <TextView android:text="@{user.name}"/>
    </LinearLayout>
</layout>

Activity:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
binding = DataBindingUtil.setContentView(this, R.layout.activity_main)
binding.user = user
binding.lifecycleOwner = this  // For LiveData

Expressions:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
@{user.name}                    - Simple
@{`Name: ` + user.name}         - Concatenation
@{user.age > 18 ? `Adult` : `Child`}  - Ternary
@{user.name ?? `Unknown`}       - Elvis
@={user.name}                   - Two-way
```

---

### When to Use What:

```
Simple App:
✅ View Binding (faster, simpler)

MVVM App:
✅ Data Binding with LiveData

Complex UI:
✅ Data Binding with BindingAdapters

Form App:
✅ Two-Way Binding

Observable Data:
✅ LiveData (best) or ObservableField
```

---

**মনে রাখো:**

```
Data Binding = View Binding + Extra Features

View Binding:
- findViewById() replacement
- Simple, fast

Data Binding:
- View Binding features +
- XML expressions @{}
- Two-way binding @={}
- Observable data
- LiveData support
- BindingAdapters
- Event handling

Choose based on need! 🎯
```

---

**Created by: Claude for Hasibuzzaman Chowdhury**  
**Date: December 31, 2025**
