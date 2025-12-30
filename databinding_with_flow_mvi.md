# 🎯 MVI with Data Binding & Flow - CORRECTED Tutorial

Complete two-way binding with StateFlow using BindingAdapter! 🚀

---

## 🤔 Why BindingAdapter is REQUIRED:

```
Problem:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

StateFlow is READ-ONLY (immutable)
↓
Can't use @={} directly (two-way binding)
↓
Need custom solution!

Solution:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

BindingAdapter + onTextChanged
↓
Simulates two-way binding
↓
Works perfectly with StateFlow!
```

---

## 📝 Step 1: Setup (Same as before)

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
    implementation "androidx.lifecycle:lifecycle-runtime-ktx:2.6.2"
    implementation "androidx.activity:activity-ktx:1.8.0"
    implementation "org.jetbrains.kotlinx:kotlinx-coroutines-android:1.7.3"
}
```

---

## 📝 Step 2: UI State (Same)

```kotlin
// UserProfileUiState.kt
package com.example.mvipractice

data class UserProfileUiState(
    val name: String = "",
    val email: String = "",
    val phone: String = "",
    val age: String = "",
    val bio: String = "",
    val isActive: Boolean = true,
    val isLoading: Boolean = false,
    val errorMessage: String? = null,
    val successMessage: String? = null,
    val isFormValid: Boolean = false
) {
    fun validate(): UserProfileUiState {
        val isValid = name.isNotEmpty() && 
                     email.isNotEmpty() && 
                     email.contains("@") &&
                     phone.isNotEmpty()
        
        return copy(isFormValid = isValid)
    }
}
```

---

## 📝 Step 3: Events (Same)

```kotlin
// UserProfileEvent.kt
package com.example.mvipractice

sealed class UserProfileEvent {
    data class NameChanged(val name: String) : UserProfileEvent()
    data class EmailChanged(val email: String) : UserProfileEvent()
    data class PhoneChanged(val phone: String) : UserProfileEvent()
    data class AgeChanged(val age: String) : UserProfileEvent()
    data class BioChanged(val bio: String) : UserProfileEvent()
    data class ActiveStatusChanged(val isActive: Boolean) : UserProfileEvent()
    
    object LoadData : UserProfileEvent()
    object SubmitForm : UserProfileEvent()
    object ClearForm : UserProfileEvent()
    object ClearError : UserProfileEvent()
    object ClearSuccess : UserProfileEvent()
}
```

---

## 📝 Step 4: ViewModel (Same)

```kotlin
// UserProfileViewModel.kt
package com.example.mvipractice

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.asStateFlow
import kotlinx.coroutines.flow.update
import kotlinx.coroutines.launch
import kotlinx.coroutines.delay
import android.util.Log

class UserProfileViewModel : ViewModel() {
    
    private val _uiState = MutableStateFlow(UserProfileUiState())
    val uiState: StateFlow<UserProfileUiState> = _uiState.asStateFlow()
    
    init {
        Log.d("ViewModel", "ViewModel created")
    }
    
    fun onEvent(event: UserProfileEvent) {
        when (event) {
            is UserProfileEvent.NameChanged -> updateName(event.name)
            is UserProfileEvent.EmailChanged -> updateEmail(event.email)
            is UserProfileEvent.PhoneChanged -> updatePhone(event.phone)
            is UserProfileEvent.AgeChanged -> updateAge(event.age)
            is UserProfileEvent.BioChanged -> updateBio(event.bio)
            is UserProfileEvent.ActiveStatusChanged -> updateActiveStatus(event.isActive)
            is UserProfileEvent.LoadData -> loadData()
            is UserProfileEvent.SubmitForm -> submitForm()
            is UserProfileEvent.ClearForm -> clearForm()
            is UserProfileEvent.ClearError -> clearError()
            is UserProfileEvent.ClearSuccess -> clearSuccess()
        }
    }
    
    private fun updateName(name: String) {
        _uiState.update { it.copy(name = name).validate() }
        Log.d("ViewModel", "Name updated: $name")
    }
    
    private fun updateEmail(email: String) {
        _uiState.update { it.copy(email = email).validate() }
        Log.d("ViewModel", "Email updated: $email")
    }
    
    private fun updatePhone(phone: String) {
        _uiState.update { it.copy(phone = phone).validate() }
        Log.d("ViewModel", "Phone updated: $phone")
    }
    
    private fun updateAge(age: String) {
        _uiState.update { it.copy(age = age).validate() }
        Log.d("ViewModel", "Age updated: $age")
    }
    
    private fun updateBio(bio: String) {
        _uiState.update { it.copy(bio = bio) }
        Log.d("ViewModel", "Bio updated: $bio")
    }
    
    private fun updateActiveStatus(isActive: Boolean) {
        _uiState.update { it.copy(isActive = isActive) }
        Log.d("ViewModel", "Active status: $isActive")
    }
    
    private fun loadData() {
        Log.d("ViewModel", "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━")
        Log.d("ViewModel", "Loading data...")
        
        viewModelScope.launch {
            _uiState.update { it.copy(isLoading = true) }
            delay(1500)
            
            _uiState.update { currentState ->
                currentState.copy(
                    name = "Hasibuzzaman Chowdhury",
                    email = "hasib@example.com",
                    phone = "01712345678",
                    age = "25",
                    bio = "Android Developer from Bangladesh",
                    isActive = true,
                    isLoading = false
                ).validate()
            }
            
            Log.d("ViewModel", "✅ Data loaded")
            Log.d("ViewModel", "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━")
        }
    }
    
    private fun submitForm() {
        Log.d("ViewModel", "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━")
        Log.d("ViewModel", "Submit clicked")
        
        val currentState = _uiState.value
        
        if (!currentState.isFormValid) {
            _uiState.update { 
                it.copy(errorMessage = "Please fill all required fields") 
            }
            Log.d("ViewModel", "❌ Validation failed")
            return
        }
        
        viewModelScope.launch {
            _uiState.update { it.copy(isLoading = true, errorMessage = null) }
            
            Log.d("ViewModel", "Submitting:")
            Log.d("ViewModel", "Name: ${currentState.name}")
            Log.d("ViewModel", "Email: ${currentState.email}")
            Log.d("ViewModel", "Phone: ${currentState.phone}")
            Log.d("ViewModel", "Age: ${currentState.age}")
            
            delay(2000)
            
            _uiState.update { 
                it.copy(
                    isLoading = false,
                    successMessage = "✅ Profile updated successfully!"
                )
            }
            
            Log.d("ViewModel", "✅ Success")
            Log.d("ViewModel", "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━")
        }
    }
    
    private fun clearForm() {
        _uiState.update { UserProfileUiState() }
        Log.d("ViewModel", "Form cleared")
    }
    
    private fun clearError() {
        _uiState.update { it.copy(errorMessage = null) }
    }
    
    private fun clearSuccess() {
        _uiState.update { it.copy(successMessage = null) }
    }
}
```

---

## 📝 Step 5: ✅ BINDING ADAPTERS (REQUIRED!)

```kotlin
// FlowBindingAdapters.kt
package com.example.mvipractice

import android.text.Editable
import android.text.TextWatcher
import android.widget.EditText
import androidx.databinding.BindingAdapter
import androidx.databinding.InverseBindingAdapter
import androidx.databinding.InverseBindingListener

object FlowBindingAdapters {
    
    // ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    // Part 1: Set value (State → View)
    // ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    
    @JvmStatic
    @BindingAdapter("flowText")
    fun setFlowText(view: EditText, text: String?) {
        // Prevent infinite loop by checking if text is different
        if (view.text.toString() != text) {
            view.setText(text)
        }
    }
    
    // ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    // Part 2: Get value (View → State)
    // ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    
    @JvmStatic
    @InverseBindingAdapter(
        attribute = "flowText",
        event = "flowTextAttrChanged"
    )
    fun getFlowText(view: EditText): String {
        return view.text.toString()
    }
    
    // ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    // Part 3: Listen for changes
    // ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    
    @JvmStatic
    @BindingAdapter("flowTextAttrChanged")
    fun setFlowTextListener(view: EditText, listener: InverseBindingListener?) {
        // Only add listener if not already added
        view.tag?.let { existingListener ->
            view.removeTextChangedListener(existingListener as TextWatcher)
        }
        
        val textWatcher = object : TextWatcher {
            override fun beforeTextChanged(s: CharSequence?, start: Int, count: Int, after: Int) {}
            
            override fun onTextChanged(s: CharSequence?, start: Int, before: Int, count: Int) {
                // Notify Data Binding that text changed
                listener?.onChange()
            }
            
            override fun afterTextChanged(s: Editable?) {}
        }
        
        view.tag = textWatcher
        view.addTextChangedListener(textWatcher)
    }
    
    // ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    // Part 4: Text change callback (for events)
    // ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    
    @JvmStatic
    @BindingAdapter("onFlowTextChanged")
    fun setOnFlowTextChanged(view: EditText, callback: ((String) -> Unit)?) {
        // Remove old listener if exists
        val oldWatcher = view.getTag(R.id.text_change_listener) as? TextWatcher
        oldWatcher?.let { view.removeTextChangedListener(it) }
        
        callback?.let {
            val textWatcher = object : TextWatcher {
                override fun beforeTextChanged(s: CharSequence?, start: Int, count: Int, after: Int) {}
                
                override fun onTextChanged(s: CharSequence?, start: Int, before: Int, count: Int) {
                    callback.invoke(s.toString())
                }
                
                override fun afterTextChanged(s: Editable?) {}
            }
            
            view.setTag(R.id.text_change_listener, textWatcher)
            view.addTextChangedListener(textWatcher)
        }
    }
}
```

---

## 📝 Step 5.5: Create IDs for tags

```xml
<!-- res/values/ids.xml -->
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <item name="text_change_listener" type="id"/>
</resources>
```

---

## 📝 Step 6: ✅ XML WITH BINDING ADAPTER

```xml
<?xml version="1.0" encoding="utf-8"?>
<!-- activity_main.xml -->

<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">
    
    <data>
        <import type="android.view.View"/>
        
        <variable
            name="uiState"
            type="com.example.mvipractice.UserProfileUiState"/>
        
        <variable
            name="viewModel"
            type="com.example.mvipractice.UserProfileViewModel"/>
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
                android:text="MVI with Flow Binding"
                android:textSize="24sp"
                android:textStyle="bold"
                android:gravity="center"
                android:layout_marginBottom="8dp"/>
            
            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="StateFlow + Custom BindingAdapter"
                android:textSize="14sp"
                android:gravity="center"
                android:layout_marginBottom="24dp"
                android:textColor="#666666"/>
            
            <!-- Loading -->
            <ProgressBar
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_gravity="center"
                android:layout_marginBottom="16dp"
                android:visibility="@{uiState.loading ? View.VISIBLE : View.GONE}"/>
            
            <!-- Error Message -->
            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="@{uiState.errorMessage}"
                android:textColor="#FF0000"
                android:textSize="14sp"
                android:padding="12dp"
                android:background="#FFEBEE"
                android:layout_marginBottom="16dp"
                android:visibility="@{uiState.errorMessage != null ? View.VISIBLE : View.GONE}"/>
            
            <!-- Success Message -->
            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="@{uiState.successMessage}"
                android:textColor="#4CAF50"
                android:textSize="14sp"
                android:padding="12dp"
                android:background="#E8F5E9"
                android:layout_marginBottom="16dp"
                android:visibility="@{uiState.successMessage != null ? View.VISIBLE : View.GONE}"/>
            
            <!-- ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ -->
            <!-- ✅ Name Input with FlowBinding -->
            <!-- ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ -->
            
            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="Name *"
                android:textStyle="bold"
                android:layout_marginBottom="4dp"/>
            
            <EditText
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:hint="Enter name"
                app:flowText="@{uiState.name}"
                app:onFlowTextChanged="@{(text) -> viewModel.onEvent(UserProfileEvent.NameChanged.INSTANCE.invoke(text))}"
                android:layout_marginBottom="16dp"/>
            
            <!-- ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ -->
            <!-- ✅ Email Input with FlowBinding -->
            <!-- ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ -->
            
            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="Email *"
                android:textStyle="bold"
                android:layout_marginBottom="4dp"/>
            
            <EditText
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:hint="Enter email"
                android:inputType="textEmailAddress"
                app:flowText="@{uiState.email}"
                app:onFlowTextChanged="@{(text) -> viewModel.onEvent(UserProfileEvent.EmailChanged.INSTANCE.invoke(text))}"
                android:layout_marginBottom="16dp"/>
            
            <!-- ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ -->
            <!-- ✅ Phone Input with FlowBinding -->
            <!-- ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ -->
            
            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="Phone *"
                android:textStyle="bold"
                android:layout_marginBottom="4dp"/>
            
            <EditText
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:hint="Enter phone"
                android:inputType="phone"
                app:flowText="@{uiState.phone}"
                app:onFlowTextChanged="@{(text) -> viewModel.onEvent(UserProfileEvent.PhoneChanged.INSTANCE.invoke(text))}"
                android:layout_marginBottom="16dp"/>
            
            <!-- ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ -->
            <!-- ✅ Age Input with FlowBinding -->
            <!-- ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ -->
            
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
                app:flowText="@{uiState.age}"
                app:onFlowTextChanged="@{(text) -> viewModel.onEvent(UserProfileEvent.AgeChanged.INSTANCE.invoke(text))}"
                android:layout_marginBottom="16dp"/>
            
            <!-- ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ -->
            <!-- ✅ Bio Input with FlowBinding -->
            <!-- ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ -->
            
            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="Bio"
                android:textStyle="bold"
                android:layout_marginBottom="4dp"/>
            
            <EditText
                android:layout_width="match_parent"
                android:layout_height="100dp"
                android:hint="Tell us about yourself"
                android:gravity="top"
                app:flowText="@{uiState.bio}"
                app:onFlowTextChanged="@{(text) -> viewModel.onEvent(UserProfileEvent.BioChanged.INSTANCE.invoke(text))}"
                android:layout_marginBottom="16dp"/>
            
            <!-- Active Status Switch -->
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="horizontal"
                android:gravity="center_vertical"
                android:layout_marginBottom="24dp">
                
                <TextView
                    android:layout_width="0dp"
                    android:layout_height="wrap_content"
                    android:layout_weight="1"
                    android:text="Active Status"
                    android:textStyle="bold"/>
                
                <androidx.appcompat.widget.SwitchCompat
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:checked="@{uiState.active}"
                    android:onCheckedChanged="@{(view, checked) -> viewModel.onEvent(UserProfileEvent.ActiveStatusChanged.INSTANCE.invoke(checked))}"/>
            </LinearLayout>
            
            <!-- Live Preview -->
            <View
                android:layout_width="match_parent"
                android:layout_height="1dp"
                android:background="#CCCCCC"
                android:layout_marginBottom="16dp"/>
            
            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="Live Preview"
                android:textSize="18sp"
                android:textStyle="bold"
                android:layout_marginBottom="8dp"/>
            
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="vertical"
                android:padding="16dp"
                android:background="#F5F5F5"
                android:layout_marginBottom="24dp">
                
                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="@{`Name: ` + uiState.name}"/>
                
                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="@{`Email: ` + uiState.email}"/>
                
                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="@{`Phone: ` + uiState.phone}"/>
                
                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="@{`Age: ` + uiState.age}"/>
                
                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="@{`Status: ` + (uiState.active ? `Active` : `Inactive`)}"/>
                
                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="@{`Valid: ` + (uiState.formValid ? `✅ Yes` : `❌ No`)}"
                    android:textStyle="bold"
                    android:layout_marginTop="8dp"/>
            </LinearLayout>
            
            <!-- Buttons -->
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="horizontal"
                android:layout_marginBottom="16dp">
                
                <Button
                    android:layout_width="0dp"
                    android:layout_height="wrap_content"
                    android:layout_weight="1"
                    android:text="Load Data"
                    android:onClick="@{() -> viewModel.onEvent(UserProfileEvent.LoadData.INSTANCE)}"
                    android:layout_marginEnd="8dp"/>
                
                <Button
                    android:layout_width="0dp"
                    android:layout_height="wrap_content"
                    android:layout_weight="1"
                    android:text="Clear"
                    android:onClick="@{() -> viewModel.onEvent(UserProfileEvent.ClearForm.INSTANCE)}"/>
            </LinearLayout>
            
            <Button
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="Submit"
                android:enabled="@{uiState.formValid &amp;&amp; !uiState.loading}"
                android:onClick="@{() -> viewModel.onEvent(UserProfileEvent.SubmitForm.INSTANCE)}"/>
            
        </LinearLayout>
    </ScrollView>
</layout>
```

---

## 📝 Step 7: MainActivity (Same)

```kotlin
// MainActivity.kt
package com.example.mvipractice

import android.os.Bundle
import android.widget.Toast
import androidx.activity.viewModels
import androidx.appcompat.app.AppCompatActivity
import androidx.databinding.DataBindingUtil
import androidx.lifecycle.Lifecycle
import androidx.lifecycle.lifecycleScope
import androidx.lifecycle.repeatOnLifecycle
import com.example.mvipractice.databinding.ActivityMainBinding
import kotlinx.coroutines.launch

class MainActivity : AppCompatActivity() {
    
    private val viewModel: UserProfileViewModel by viewModels()
    private lateinit var binding: ActivityMainBinding
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        binding = DataBindingUtil.setContentView(this, R.layout.activity_main)
        binding.viewModel = viewModel
        binding.lifecycleOwner = this
        
        observeUiState()
    }
    
    private fun observeUiState() {
        lifecycleScope.launch {
            repeatOnLifecycle(Lifecycle.State.STARTED) {
                viewModel.uiState.collect { state ->
                    binding.uiState = state
                    handleSideEffects(state)
                }
            }
        }
    }
    
    private fun handleSideEffects(state: UserProfileUiState) {
        state.errorMessage?.let { error ->
            Toast.makeText(this, error, Toast.LENGTH_SHORT).show()
            viewModel.onEvent(UserProfileEvent.ClearError)
        }
        
        state.successMessage?.let { success ->
            Toast.makeText(this, success, Toast.LENGTH_LONG).show()
            viewModel.onEvent(UserProfileEvent.ClearSuccess)
        }
    }
}
```

---

## 🎯 How It Works Now:

```
Complete Two-Way Flow with BindingAdapter:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Direction 1: State → View
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. State changes: uiState.name = "Hasib"
2. Data Binding detects change
3. Calls: setFlowText(editText, "Hasib")
4. EditText displays: "Hasib"

Direction 2: View → State
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. User types: "New Name"
2. TextWatcher detects (from onFlowTextChanged)
3. Calls: callback("New Name")
4. Triggers: onEvent(NameChanged("New Name"))
5. ViewModel updates: _uiState.update { copy(name = "New Name") }
6. State flows back to View (Direction 1)

Magic happens! ✨
```

---

## 📊 XML Attribute Breakdown:

```xml
<EditText
    app:flowText="@{uiState.name}"
    app:onFlowTextChanged="@{(text) -> viewModel.onEvent(...)}"/>
```

**Explanation:**
```
app:flowText="@{uiState.name}"
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Display value from state (one-way)
↓
setFlowText() called
↓
EditText.setText("Hasib")

app:onFlowTextChanged="@{...}"
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Listen for user input
↓
TextWatcher detects change
↓
Callback triggered
↓
onEvent(NameChanged("New Name"))
↓
State updated

Together = Two-way binding! ✨
```

---

## 🎓 Summary:

```
BindingAdapter Required Because:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

StateFlow is immutable
↓
Can't use @={} directly
↓
Need custom solution
↓
BindingAdapter bridges the gap!

Components Needed:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

1. flowText (display value)
   @BindingAdapter("flowText")
   
2. onFlowTextChanged (listen changes)
   @BindingAdapter("onFlowTextChanged")
   
3. XML uses both:
   app:flowText="@{state}"
   app:onFlowTextChanged="@{callback}"

Result:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Perfect two-way binding with StateFlow!
```

---

**Created by: Claude for Hasibuzzaman Chowdhury**  
**Date: December 31, 2025**

**MVI + StateFlow + BindingAdapter = True Two-Way Binding! 🎯🚀**
