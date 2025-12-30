# 🎯 View Binding - Step by Step Complete Tutorial

একদম scratch থেকে View Binding শিখি! 🚀

---

## 📖 Table of Contents

1. [View Binding কি?](#view-binding-কি)
2. [কেন দরকার?](#কেন-দরকার)
3. [Step 1: Project Setup](#step-1-project-setup)
4. [Step 2: Create Layout](#step-2-create-layout)
5. [Step 3: Old Way (Without View Binding)](#step-3-old-way-without-view-binding)
6. [Step 4: New Way (With View Binding)](#step-4-new-way-with-view-binding)
7. [Complete Working Example](#complete-working-example)
8. [Naming Convention](#naming-convention)
9. [Best Practices](#best-practices)
10. [Summary](#summary)

---

## 🤔 View Binding কি?

### সহজ ভাষায়:

```
View Binding = "XML layout থেকে automatically type-safe code generate করা"

Before View Binding:
❌ findViewById() লিখতে হতো
❌ Null pointer exception হতে পারতো
❌ Type safety ছিল না

After View Binding:
✅ Automatically binding class তৈরি হয়
✅ Null-safe
✅ Type-safe
✅ findViewById() দরকার নেই!
```

---

## 🎯 কেন দরকার?

### Problem with findViewById():

```kotlin
// ❌ Old Way - findViewById()
val textView = findViewById<TextView>(R.id.tvTitle)
val button = findViewById<Button>(R.id.btnSubmit)

Problems:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
❌ Lots of boilerplate code
❌ Null pointer exceptions possible
❌ Not type-safe (wrong cast = crash!)
❌ Runtime overhead
❌ ID typo = crash at runtime
❌ Repetitive code
```

### Solution - View Binding:

```kotlin
// ✅ New Way - View Binding
binding.tvTitle.text = "Hello"
binding.btnSubmit.setOnClickListener { }

Benefits:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ No boilerplate
✅ Null-safe
✅ Type-safe
✅ Compile-time checking
✅ Fast (no runtime lookup)
✅ Auto-complete works perfectly
✅ Refactor-safe
```

---

## 📝 Step 1: Project Setup

### 1.1 Enable View Binding

Open **app/build.gradle** (Module level):

```gradle
android {
    ...
    
    // ✅ Add this inside android block
    buildFeatures {
        viewBinding = true
    }
}
```

### 1.2 Sync Project

Click **"Sync Now"** at the top right.

### 1.3 What Happens?

```
After enabling View Binding:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

For each XML layout file, Android Studio generates:
- A binding class automatically
- Type-safe view references
- No runtime overhead

Example:
activity_main.xml → ActivityMainBinding.class
fragment_profile.xml → FragmentProfileBinding.class
item_user.xml → ItemUserBinding.class

Pattern: PascalCase + "Binding"
```

---

## 📝 Step 2: Create Layout

Create **activity_main.xml**:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp"
    android:gravity="center">
    
    <!-- Title -->
    <TextView
        android:id="@+id/tvTitle"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="View Binding Example"
        android:textSize="24sp"
        android:textStyle="bold"
        android:gravity="center"
        android:layout_marginBottom="32dp"/>
    
    <!-- Name Input -->
    <EditText
        android:id="@+id/etName"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Enter your name"
        android:layout_marginBottom="16dp"/>
    
    <!-- Email Input -->
    <EditText
        android:id="@+id/etEmail"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Enter your email"
        android:inputType="textEmailAddress"
        android:layout_marginBottom="16dp"/>
    
    <!-- Age Input -->
    <EditText
        android:id="@+id/etAge"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Enter your age"
        android:inputType="number"
        android:layout_marginBottom="16dp"/>
    
    <!-- Submit Button -->
    <Button
        android:id="@+id/btnSubmit"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Submit"
        android:layout_marginBottom="16dp"/>
    
    <!-- Clear Button -->
    <Button
        android:id="@+id/btnClear"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Clear"/>
    
    <!-- Result TextView -->
    <TextView
        android:id="@+id/tvResult"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="32dp"
        android:textSize="16sp"
        android:gravity="center"/>
    
</LinearLayout>
```

### What Views We Have:

```
View IDs in XML:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ tvTitle - TextView (title)
✅ etName - EditText (name input)
✅ etEmail - EditText (email input)
✅ etAge - EditText (age input)
✅ btnSubmit - Button (submit)
✅ btnClear - Button (clear)
✅ tvResult - TextView (show result)
```

---

## 📝 Step 3: Old Way (Without View Binding)

### ❌ The Problem:

```kotlin
class MainActivity : AppCompatActivity() {
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        // ❌ findViewById for EVERY view (Boilerplate!)
        val tvTitle = findViewById<TextView>(R.id.tvTitle)
        val etName = findViewById<EditText>(R.id.etName)
        val etEmail = findViewById<EditText>(R.id.etEmail)
        val etAge = findViewById<EditText>(R.id.etAge)
        val btnSubmit = findViewById<Button>(R.id.btnSubmit)
        val btnClear = findViewById<Button>(R.id.btnClear)
        val tvResult = findViewById<TextView>(R.id.tvResult)
        
        // Problems:
        // ❌ 7 lines just to find views!
        // ❌ Lots of boilerplate code
        // ❌ Null pointer exceptions possible
        // ❌ Not type-safe (wrong cast = crash!)
        // ❌ Runtime performance hit
        // ❌ Typo in ID = crash at runtime
        
        // Use views
        btnSubmit.setOnClickListener {
            val name = etName.text.toString()
            val email = etEmail.text.toString()
            val age = etAge.text.toString()
            
            if (name.isNotEmpty() && email.isNotEmpty() && age.isNotEmpty()) {
                tvResult.text = "Name: $name\nEmail: $email\nAge: $age"
            }
        }
        
        btnClear.setOnClickListener {
            etName.text.clear()
            etEmail.text.clear()
            etAge.text.clear()
            tvResult.text = ""
        }
    }
}
```

### Count the Problems:

```
Problems with findViewById():
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
❌ 7 views = 7 lines of findViewById
❌ Repetitive boilerplate code
❌ Null safety না থাকা
❌ Type safety না থাকা
❌ Runtime performance issue
❌ Refactoring difficult
❌ ID rename করলে manually change করতে হয়
❌ Wrong type cast = crash!

Example crash:
val button = findViewById<Button>(R.id.tvTitle) // TextView কে Button cast!
button.setOnClickListener { } // 💥 ClassCastException!
```

---

## 📝 Step 4: New Way (With View Binding)

### ✅ The Solution:

```kotlin
class MainActivity : AppCompatActivity() {
    
    // ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    // Step 4.1: Declare binding variable
    // ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    private lateinit var binding: ActivityMainBinding
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
        // Step 4.2: Inflate binding
        // ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
        binding = ActivityMainBinding.inflate(layoutInflater)
        
        // ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
        // Step 4.3: Set content view with binding.root
        // ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
        setContentView(binding.root)
        
        // ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
        // Step 4.4: Access views directly - No findViewById!
        // ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
        
        // ✅ Change title
        binding.tvTitle.text = "User Registration"
        
        // ✅ Submit button click
        binding.btnSubmit.setOnClickListener {
            // Get input values
            val name = binding.etName.text.toString()
            val email = binding.etEmail.text.toString()
            val age = binding.etAge.text.toString()
            
            // Validate
            if (name.isEmpty()) {
                binding.etName.error = "Name required"
                return@setOnClickListener
            }
            
            if (email.isEmpty()) {
                binding.etEmail.error = "Email required"
                return@setOnClickListener
            }
            
            if (age.isEmpty()) {
                binding.etAge.error = "Age required"
                return@setOnClickListener
            }
            
            // Show result
            binding.tvResult.text = """
                ✅ Registration Successful!
                
                Name: $name
                Email: $email
                Age: $age years old
            """.trimIndent()
            
            Toast.makeText(this, "Submitted!", Toast.LENGTH_SHORT).show()
        }
        
        // ✅ Clear button click
        binding.btnClear.setOnClickListener {
            binding.etName.text.clear()
            binding.etEmail.text.clear()
            binding.etAge.text.clear()
            binding.tvResult.text = ""
            
            Toast.makeText(this, "Cleared!", Toast.LENGTH_SHORT).show()
        }
    }
}
```

### What Changed?

```
Before (findViewById):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
setContentView(R.layout.activity_main)

val tvTitle = findViewById<TextView>(R.id.tvTitle)
val etName = findViewById<EditText>(R.id.etName)
val btnSubmit = findViewById<Button>(R.id.btnSubmit)
// ... 4 more findViewById calls

tvTitle.text = "Hello"
etName.setText("John")
btnSubmit.setOnClickListener { }

Lines of code: 10+ lines just for views!


After (View Binding):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
binding = ActivityMainBinding.inflate(layoutInflater)
setContentView(binding.root)

binding.tvTitle.text = "Hello"
binding.etName.setText("John")
binding.btnSubmit.setOnClickListener { }

Lines of code: 2 lines setup + direct access!
```

---

## 💻 Complete Working Example

### MainActivity.kt - Full Implementation:

```kotlin
package com.example.viewbindingexample

import android.graphics.Color
import android.os.Bundle
import android.util.Log
import android.widget.Toast
import androidx.appcompat.app.AppCompatActivity
import com.example.viewbindingexample.databinding.ActivityMainBinding
import java.text.SimpleDateFormat
import java.util.*

class MainActivity : AppCompatActivity() {
    
    // ✅ Declare binding
    private lateinit var binding: ActivityMainBinding
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        Log.d("MainActivity", "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━")
        Log.d("MainActivity", "Step 1: Activity created")
        
        // ✅ Inflate binding
        binding = ActivityMainBinding.inflate(layoutInflater)
        Log.d("MainActivity", "Step 2: Binding inflated")
        Log.d("MainActivity", "Binding class: ${binding::class.simpleName}")
        
        // ✅ Set content view
        setContentView(binding.root)
        Log.d("MainActivity", "Step 3: Content view set")
        
        // ✅ Setup UI
        setupViews()
        Log.d("MainActivity", "Step 4: Views setup complete")
        Log.d("MainActivity", "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━")
    }
    
    private fun setupViews() {
        // Change title color
        binding.tvTitle.setTextColor(Color.BLUE)
        
        // Submit button
        binding.btnSubmit.setOnClickListener {
            Log.d("MainActivity", "Submit button clicked")
            handleSubmit()
        }
        
        // Clear button
        binding.btnClear.setOnClickListener {
            Log.d("MainActivity", "Clear button clicked")
            handleClear()
        }
    }
    
    private fun handleSubmit() {
        // Get values
        val name = binding.etName.text.toString()
        val email = binding.etEmail.text.toString()
        val age = binding.etAge.text.toString()
        
        Log.d("Submit", "Name: $name")
        Log.d("Submit", "Email: $email")
        Log.d("Submit", "Age: $age")
        
        // Validate
        when {
            name.isEmpty() -> {
                binding.etName.error = "Name required"
                binding.etName.requestFocus()
                Log.d("Submit", "Validation failed: Name empty")
                return
            }
            email.isEmpty() -> {
                binding.etEmail.error = "Email required"
                binding.etEmail.requestFocus()
                Log.d("Submit", "Validation failed: Email empty")
                return
            }
            !email.contains("@") -> {
                binding.etEmail.error = "Invalid email"
                binding.etEmail.requestFocus()
                Log.d("Submit", "Validation failed: Invalid email")
                return
            }
            age.isEmpty() -> {
                binding.etAge.error = "Age required"
                binding.etAge.requestFocus()
                Log.d("Submit", "Validation failed: Age empty")
                return
            }
            age.toIntOrNull() == null || age.toInt() <= 0 -> {
                binding.etAge.error = "Age must be a valid number"
                binding.etAge.requestFocus()
                Log.d("Submit", "Validation failed: Invalid age")
                return
            }
        }
        
        // Show result
        binding.tvResult.text = """
            ✅ Registration Successful!
            
            👤 Name: $name
            📧 Email: $email
            🎂 Age: $age years old
            📅 Registered: ${getCurrentDateTime()}
        """.trimIndent()
        
        binding.tvResult.setTextColor(Color.GREEN)
        
        Toast.makeText(this, "✅ Submitted successfully!", Toast.LENGTH_SHORT).show()
        
        Log.d("Submit", "✅ Validation passed - Data submitted")
    }
    
    private fun handleClear() {
        binding.etName.text.clear()
        binding.etEmail.text.clear()
        binding.etAge.text.clear()
        binding.tvResult.text = ""
        binding.tvResult.setTextColor(Color.BLACK)
        
        Toast.makeText(this, "🗑️ Cleared!", Toast.LENGTH_SHORT).show()
        
        Log.d("Clear", "All fields cleared")
    }
    
    private fun getCurrentDateTime(): String {
        val format = SimpleDateFormat("dd MMM yyyy, hh:mm a", Locale.getDefault())
        return format.format(Date())
    }
}
```

---

## 📊 Comparison Table

| Feature | findViewById (❌) | View Binding (✅) |
|---------|------------------|-------------------|
| **Boilerplate** | ❌ Lots | ✅ Minimal |
| **Type Safety** | ❌ No | ✅ Yes |
| **Null Safety** | ❌ No | ✅ Yes |
| **Performance** | ❌ Runtime lookup | ✅ Compile-time |
| **Auto-complete** | ⚠️ Limited | ✅ Full support |
| **Refactor** | ❌ Manual | ✅ Automatic |
| **Crash Risk** | ❌ High | ✅ Low |
| **Code Lines** | ❌ Many | ✅ Few |

---

## 📊 Behind the Scenes

### How View Binding Works:

```
Step 1: You create XML
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
activity_main.xml

<TextView android:id="@+id/tvTitle" />
<EditText android:id="@+id/etName" />
<Button android:id="@+id/btnSubmit" />

↓ (Build time)

Step 2: Android generates binding class
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
ActivityMainBinding.java (Auto-generated)

public final class ActivityMainBinding {
    public final LinearLayout root;
    public final TextView tvTitle;
    public final EditText etName;
    public final Button btnSubmit;
    
    public static ActivityMainBinding inflate(LayoutInflater inflater) {
        // Inflate layout and find all views
        // Return binding with all views initialized
    }
}

↓ (You use)

Step 3: You use in your code
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
binding = ActivityMainBinding.inflate(layoutInflater)
binding.tvTitle.text = "Hello"  // ✅ Direct access!
binding.btnSubmit.setOnClickListener { }  // ✅ Type-safe!
```

---

## 🎯 Naming Convention

### XML Layout to Binding Class:

```
XML Layout File               →  Binding Class Name
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
activity_main.xml             →  ActivityMainBinding
activity_user_profile.xml     →  ActivityUserProfileBinding
fragment_settings.xml         →  FragmentSettingsBinding
fragment_home_screen.xml      →  FragmentHomeScreenBinding
dialog_confirmation.xml       →  DialogConfirmationBinding
item_user.xml                 →  ItemUserBinding
layout_header.xml             →  LayoutHeaderBinding
custom_view_player.xml        →  CustomViewPlayerBinding

Pattern: 
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
snake_case.xml → PascalCaseBinding

Rule:
1. Remove underscores
2. Capitalize each word
3. Add "Binding" suffix
```

### View ID to Property Name:

```
XML View ID                   →  Binding Property
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
android:id="@+id/tvTitle"     →  binding.tvTitle
android:id="@+id/etName"      →  binding.etName
android:id="@+id/btnSubmit"   →  binding.btnSubmit
android:id="@+id/ivProfile"   →  binding.ivProfile
android:id="@+id/rvUserList"  →  binding.rvUserList

Rule:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
View ID এর নাম exactly same থাকে!
@+id/camelCase → binding.camelCase
```

---

## 💡 Best Practices

### ✅ DO:

```kotlin
// ✅ Use lateinit for Activity
private lateinit var binding: ActivityMainBinding

// ✅ Inflate in onCreate
binding = ActivityMainBinding.inflate(layoutInflater)

// ✅ Set content view with binding.root
setContentView(binding.root)

// ✅ Access views via binding
binding.tvTitle.text = "Hello"

// ✅ Use descriptive view IDs
android:id="@+id/tvUserName"  // Good
android:id="@+id/textView1"   // Bad

// ✅ Enable for all modules
buildFeatures {
    viewBinding = true
}
```

### ❌ DON'T:

```kotlin
// ❌ Don't use nullable for Activity
private var binding: ActivityMainBinding? = null  // Don't do this!

// ❌ Don't forget to inflate
// binding is null here - crash!

// ❌ Don't mix findViewById with View Binding
val button = findViewById<Button>(R.id.btnSubmit)  // Don't!
binding.btnSubmit.setOnClickListener { }  // Use this!

// ❌ Don't use generic IDs
android:id="@+id/button1"  // Bad
android:id="@+id/btnSubmit"  // Good
```

---

## 🎯 Fragment Example (Bonus)

### Fragment with View Binding:

```kotlin
class ProfileFragment : Fragment() {
    
    // ✅ Nullable binding for Fragment
    private var _binding: FragmentProfileBinding? = null
    
    // ✅ Non-null property for easier access
    private val binding get() = _binding!!
    
    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View {
        // ✅ Inflate binding
        _binding = FragmentProfileBinding.inflate(inflater, container, false)
        return binding.root
    }
    
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        
        // ✅ Setup UI
        binding.tvName.text = "Hasibuzzaman"
        binding.btnEdit.setOnClickListener {
            // Handle click
        }
    }
    
    override fun onDestroyView() {
        super.onDestroyView()
        
        // ✅ IMPORTANT: Clear binding to avoid memory leak!
        _binding = null
    }
}
```

### Why Nullable in Fragment?

```
Activity vs Fragment:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Activity:
- onCreate() called once
- View exists until Activity destroyed
- Use: lateinit var binding

Fragment:
- onCreateView() can be called multiple times
- View destroyed but Fragment still alive
- onDestroyView() → View gone but Fragment alive
- Must clear binding to avoid memory leak
- Use: var _binding: Type? = null
```

---

## 🎓 Summary

### View Binding in 3 Steps:

```kotlin
// Step 1: Declare
private lateinit var binding: ActivityMainBinding

// Step 2: Inflate and set
binding = ActivityMainBinding.inflate(layoutInflater)
setContentView(binding.root)

// Step 3: Use
binding.tvTitle.text = "Hello"
binding.btnSubmit.setOnClickListener { }
```

### Benefits Summary:

```
View Binding Benefits:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ No findViewById boilerplate
✅ Null-safe (no NullPointerException)
✅ Type-safe (compile-time checking)
✅ Fast (no runtime overhead)
✅ Auto-complete works perfectly
✅ Refactor-safe (rename works)
✅ Less error-prone
✅ Cleaner code
✅ Better developer experience
```

### Comparison:

```
findViewById():
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
❌ 10 lines for 10 views
❌ Repetitive code
❌ Null pointer risk
❌ Not type-safe
❌ Runtime lookup
❌ Manual refactoring

View Binding:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ 2 lines setup
✅ Direct access
✅ Null-safe
✅ Type-safe
✅ Compile-time
✅ Auto-refactoring
```

### When to Use:

```
✅ Use View Binding when:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ New projects
✅ Want type safety
✅ Want null safety
✅ Want cleaner code
✅ All Activities/Fragments

❌ Don't use when:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
❌ Need two-way data binding (use Data Binding)
❌ Legacy project with findViewById everywhere
```

---

## 📝 Quick Reference Card

```
┌─────────────────────────────────────────────┐
│         View Binding Cheat Sheet            │
├─────────────────────────────────────────────┤
│                                             │
│ 1. Enable (build.gradle):                  │
│    buildFeatures { viewBinding = true }    │
│                                             │
│ 2. Activity:                                │
│    private lateinit var binding: ...        │
│    binding = ...Binding.inflate(...)        │
│    setContentView(binding.root)             │
│                                             │
│ 3. Fragment:                                │
│    private var _binding: ...? = null        │
│    private val binding get() = _binding!!   │
│    _binding = ...Binding.inflate(...)       │
│    return binding.root                      │
│    _binding = null (in onDestroyView)       │
│                                             │
│ 4. Usage:                                   │
│    binding.viewId.property                  │
│    binding.tvTitle.text = "Hello"           │
│    binding.btnSubmit.setOnClickListener { } │
│                                             │
└─────────────────────────────────────────────┘
```

---

## 🎯 Final Tips

```
1. Always enable View Binding in new projects
2. Use lateinit for Activity, nullable for Fragment
3. Don't forget to clear Fragment binding
4. Use descriptive view IDs
5. Never mix findViewById with View Binding
6. Enjoy cleaner, safer code! 🎉
```

---

**মনে রাখো:**
```
View Binding = findViewById() এর modern replacement

Old: findViewById<TextView>(R.id.tvTitle).text = "Hello"
New: binding.tvTitle.text = "Hello"

3 Steps:
1. Declare binding
2. Inflate binding + setContentView(binding.root)
3. Use: binding.viewId.property

That's it! Super simple! 🚀
```

---

**Created by: Claude for Hasibuzzaman Chowdhury**  
**Date: December 31, 2025**
