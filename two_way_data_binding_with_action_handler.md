# 🎯 Simple Two-Way Data Binding - Practice Tutorial

একদম simple! শুধু two-way binding practice করার জন্য! 🚀

---

## 📝 Step 1: Enable Data Binding

```gradle
// app/build.gradle

android {
    ...
    
    buildFeatures {
        dataBinding = true
    }
}

dependencies {
    implementation "androidx.lifecycle:lifecycle-viewmodel-ktx:2.6.2"
    implementation "androidx.lifecycle:lifecycle-livedata-ktx:2.6.2"
    implementation "androidx.activity:activity-ktx:1.8.0"
}
```

---

## 📝 Step 2: Create User Data Class

```kotlin
// User.kt
package com.example.databindingpractice

data class User(
    val id: Int = 0,
    var name: String = "",
    var email: String = "",
    var phone: String = "",
    var age: String = "",
    var bio: String = ""
)
```

---

## 📝 Step 3: Create Simple ViewModel

```kotlin
// UserViewModel.kt
package com.example.databindingpractice

import androidx.lifecycle.ViewModel
import androidx.lifecycle.MutableLiveData
import android.util.Log

class UserViewModel : ViewModel() {
    
    // ✅ LiveData for two-way binding
    val name = MutableLiveData<String>()
    val email = MutableLiveData<String>()
    val phone = MutableLiveData<String>()
    val age = MutableLiveData<String>()
    val bio = MutableLiveData<String>()
    
    init {
        Log.d("ViewModel", "ViewModel created")
    }
    
    // ✅ Load initial data
    fun loadData() {
        Log.d("ViewModel", "Loading data...")
        
        name.value = "Hasibuzzaman Chowdhury"
        email.value = "hasib@example.com"
        phone.value = "01712345678"
        age.value = "25"
        bio.value = "Android Developer"
        
        Log.d("ViewModel", "Data loaded")
    }
    
    // ✅ Submit data
    fun submitData() {
        Log.d("ViewModel", "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━")
        Log.d("ViewModel", "Submit clicked!")
        Log.d("ViewModel", "Name: ${name.value}")
        Log.d("ViewModel", "Email: ${email.value}")
        Log.d("ViewModel", "Phone: ${phone.value}")
        Log.d("ViewModel", "Age: ${age.value}")
        Log.d("ViewModel", "Bio: ${bio.value}")
        Log.d("ViewModel", "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━")
    }
}
```

---

## 📝 Step 4: Create XML Layout

```xml
<?xml version="1.0" encoding="utf-8"?>
<!-- activity_main.xml -->

<layout xmlns:android="http://schemas.android.com/apk/res/android">
    
    <!-- ✅ Define ViewModel -->
    <data>
        <variable
            name="viewModel"
            type="com.example.databindingpractice.UserViewModel"/>
    </data>
    
    <ScrollView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:fillViewport="true">
        
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            android:padding="16dp">
            
            <!-- Title -->
            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="Two-Way Binding Practice"
                android:textSize="24sp"
                android:textStyle="bold"
                android:gravity="center"
                android:layout_marginBottom="32dp"/>
            
            <!-- Name Input -->
            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="Name"
                android:textStyle="bold"
                android:layout_marginBottom="4dp"/>
            
            <EditText
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:hint="Enter name"
                android:text="@={viewModel.name}"
                android:layout_marginBottom="16dp"/>
            
            <!-- Email Input -->
            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="Email"
                android:textStyle="bold"
                android:layout_marginBottom="4dp"/>
            
            <EditText
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:hint="Enter email"
                android:inputType="textEmailAddress"
                android:text="@={viewModel.email}"
                android:layout_marginBottom="16dp"/>
            
            <!-- Phone Input -->
            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="Phone"
                android:textStyle="bold"
                android:layout_marginBottom="4dp"/>
            
            <EditText
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:hint="Enter phone"
                android:inputType="phone"
                android:text="@={viewModel.phone}"
                android:layout_marginBottom="16dp"/>
            
            <!-- Age Input -->
            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="Age"
                android:textStyle="bold"
                android:layout_marginBottom="4dp"/>
            
            <EditText
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:hint="Enter age"
                android:inputType="number"
                android:text="@={viewModel.age}"
                android:layout_marginBottom="16dp"/>
            
            <!-- Bio Input -->
            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="Bio"
                android:textStyle="bold"
                android:layout_marginBottom="4dp"/>
            
            <EditText
                android:layout_width="match_parent"
                android:layout_height="100dp"
                android:hint="Enter bio"
                android:gravity="top"
                android:text="@={viewModel.bio}"
                android:layout_marginBottom="32dp"/>
            
            <!-- Live Preview Section -->
            <View
                android:layout_width="match_parent"
                android:layout_height="1dp"
                android:background="#CCCCCC"
                android:layout_marginBottom="16dp"/>
            
            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="Live Preview"
                android:textSize="20sp"
                android:textStyle="bold"
                android:layout_marginBottom="16dp"/>
            
            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="@{`Name: ` + viewModel.name}"
                android:textSize="16sp"
                android:layout_marginBottom="8dp"/>
            
            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="@{`Email: ` + viewModel.email}"
                android:textSize="16sp"
                android:layout_marginBottom="8dp"/>
            
            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="@{`Phone: ` + viewModel.phone}"
                android:textSize="16sp"
                android:layout_marginBottom="8dp"/>
            
            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="@{`Age: ` + viewModel.age}"
                android:textSize="16sp"
                android:layout_marginBottom="8dp"/>
            
            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="@{`Bio: ` + viewModel.bio}"
                android:textSize="16sp"
                android:layout_marginBottom="32dp"/>
            
            <!-- Submit Button -->
            <Button
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="Submit"
                android:onClick="@{() -> viewModel.submitData()}"/>
            
        </LinearLayout>
    </ScrollView>
</layout>
```

---

## 📝 Step 5: Create MainActivity

```kotlin
// MainActivity.kt
package com.example.databindingpractice

import android.os.Bundle
import androidx.activity.viewModels
import androidx.appcompat.app.AppCompatActivity
import androidx.databinding.DataBindingUtil
import com.example.databindingpractice.databinding.ActivityMainBinding

class MainActivity : AppCompatActivity() {
    
    // ✅ Get ViewModel
    private val viewModel: UserViewModel by viewModels()
    
    // ✅ Declare binding
    private lateinit var binding: ActivityMainBinding
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // ✅ Setup Data Binding
        binding = DataBindingUtil.setContentView(this, R.layout.activity_main)
        
        // ✅ Connect ViewModel
        binding.viewModel = viewModel
        
        // ✅ Set lifecycle owner (IMPORTANT for LiveData!)
        binding.lifecycleOwner = this
        
        // ✅ Load initial data
        viewModel.loadData()
    }
}
```

---

## 🎯 How It Works:

```
1. App Starts:
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   MainActivity created
   ↓
   Data Binding setup
   ↓
   binding.viewModel = viewModel
   binding.lifecycleOwner = this
   ↓
   viewModel.loadData() called
   ↓
   Form shows: "Hasibuzzaman Chowdhury", etc.

2. User Types:
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   User types "New Name" in EditText
   ↓
   Two-way binding (@={}) updates ViewModel
   ↓
   viewModel.name.value = "New Name"
   ↓
   Live Preview automatically updates
   ↓
   Shows "Name: New Name"

3. User Clicks Submit:
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   Button clicked
   ↓
   android:onClick="@{() -> viewModel.submitData()}"
   ↓
   ViewModel method called
   ↓
   Logs all data to Logcat
```

---

## 📊 Log Output:

```
When App Runs:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

D/ViewModel: ViewModel created
D/ViewModel: Loading data...
D/ViewModel: Data loaded

[User edits name and clicks Submit]

D/ViewModel: ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
D/ViewModel: Submit clicked!
D/ViewModel: Name: New Name
D/ViewModel: Email: hasib@example.com
D/ViewModel: Phone: 01712345678
D/ViewModel: Age: 25
D/ViewModel: Bio: Android Developer
D/ViewModel: ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 🎯 Key Points:

```
Two-Way Binding:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

XML:
android:text="@={viewModel.name}"
            ↑
         @={} means two-way!

What Happens:
1. viewModel.name changes → EditText updates
2. EditText changes → viewModel.name updates

Both directions automatically! No code needed!

One-Way Binding (for preview):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

XML:
android:text="@{`Name: ` + viewModel.name}"
            ↑
         @{} means one-way!

What Happens:
- viewModel.name changes → TextView updates
- TextView can't change viewModel.name
```

---

## 💡 Simple Summary:

```
3 Simple Steps:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

1. ViewModel:
   val name = MutableLiveData<String>()

2. XML:
   android:text="@={viewModel.name}"
                ↑
             Two-way!

3. Activity:
   binding.viewModel = viewModel
   binding.lifecycleOwner = this

That's it! Two-way binding working! 🎉
```

---

## 🎓 Practice Exercise:

```
Try This:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

1. Run the app
2. Type your name in the Name field
3. See the Live Preview update automatically!
4. Edit all fields
5. Click Submit
6. Check Logcat for your data

Experiment:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

1. Add more fields (address, city, etc)
2. Add more LiveData in ViewModel
3. Add more EditTexts with @={}
4. See live preview update automatically!
```

---

**Created by: Claude for Hasibuzzaman Chowdhury**  
**Date: December 31, 2025**

**Simple Two-Way Binding! শুধু practice এর জন্য! No validation, no complexity! 🎯✨**
