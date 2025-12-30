# 📱 Android Activity Lifecycle - সম্পূর্ণ বাংলা Tutorial

## 🤔 Activity Lifecycle কি?

### সহজ ভাষায়:
```
Activity Lifecycle = একটা Activity এর জন্ম থেকে মৃত্যু পর্যন্ত যাত্রা

মানুষের জীবনের মতো:
জন্ম → শৈশব → যৌবন → বৃদ্ধ → মৃত্যু

Activity এর জীবন:
Created → Started → Resumed → Paused → Stopped → Destroyed
```

### কেন এটা গুরুত্বপূর্ণ?
```
❌ Lifecycle না জানলে:
- App crash হবে
- Data lost হবে
- Memory leak হবে
- Battery drain হবে
- User experience খারাপ হবে

✅ Lifecycle জানলে:
- Smooth app experience
- Data saved থাকবে
- Memory efficient app
- Battery friendly
- Professional developer হবে!
```

---

## 🎯 Activity Lifecycle States - 7টি অবস্থা

```
1. onCreate()    - জন্ম (Birth)
2. onStart()     - দৃশ্যমান হওয়া শুরু (Becoming Visible)
3. onResume()    - সক্রিয় (Active/Running)
4. onPause()     - আংশিক বিরতি (Partially Hidden)
5. onStop()      - সম্পূর্ণ বিরতি (Fully Hidden)
6. onRestart()   - পুনরায় শুরু (Restarting)
7. onDestroy()   - মৃত্যু (Death)
```

---

## 📊 Visual Diagram - Activity Lifecycle

```
                    App Launch
                        ↓
                   ┌─────────┐
                   │onCreate()│  ← Activity তৈরি হচ্ছে
                   └─────────┘
                        ↓
                   ┌─────────┐
                   │onStart() │  ← User দেখতে পাচ্ছে
                   └─────────┘
                        ↓
                   ┌──────────┐
                   │onResume()│  ← User interact করতে পারছে
                   └──────────┘
                        ↓
                   ╔══════════╗
                   ║ RUNNING  ║  ← Activity সক্রিয়!
                   ╚══════════╝
                        ↓
         ┌──────────────┴──────────────┐
         │                             │
    Phone Call আসলো              Home Button চাপলো
    Dialog দেখালো                Another Activity open
         │                             │
         ↓                             ↓
    ┌─────────┐                  ┌─────────┐
    │onPause() │                  │onPause() │
    └─────────┘                  └─────────┘
         │                             │
    User ফিরে এলো                     ↓
         │                        ┌─────────┐
         ↓                        │onStop()  │  ← Activity দেখা যাচ্ছে না
    ┌──────────┐                 └─────────┘
    │onResume()│                       │
    └──────────┘                       │
         │                        User ফিরে এলো
         │                             │
         ↓                             ↓
    ╔══════════╗                 ┌───────────┐
    ║ RUNNING  ║                 │onRestart()│
    ╚══════════╝                 └───────────┘
                                       ↓
                                  ┌─────────┐
                                  │onStart() │
                                  └─────────┘
                                       ↓
                                  ┌──────────┐
                                  │onResume()│
                                  └──────────┘
                                       ↓
                                  ╔══════════╗
                                  ║ RUNNING  ║
                                  ╚══════════╝

                   Back Button চাপলো
                   Finish() call করলো
                           ↓
                      ┌──────────┐
                      │onDestroy()│  ← Activity শেষ!
                      └──────────┘
```

---

## 🎓 প্রতিটা Method বিস্তারিত

### 1️⃣ onCreate() - Activity এর জন্ম

**কখন call হয়:**
```
✅ Activity প্রথমবার তৈরি হওয়ার সময়
✅ শুধু একবার call হয় (জন্ম তো একবারই!)
```

**কি করতে হয়:**
```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    
    // 1. Layout set করো
    setContentView(R.layout.activity_main)
    
    // 2. View initialization
    val textView = findViewById<TextView>(R.id.textView)
    val button = findViewById<Button>(R.id.button)
    
    // 3. ViewModel initialize
    viewModel = ViewModelProvider(this)[MainViewModel::class.java]
    
    // 4. Click listeners set করো
    button.setOnClickListener {
        // Handle click
    }
    
    // 5. Intent থেকে data নাও (যদি থাকে)
    val userId = intent.getStringExtra("USER_ID")
    
    // 6. Saved state restore করো (screen rotate এর পর)
    savedInstanceState?.let {
        val savedText = it.getString("KEY_TEXT")
    }
    
    Log.d("Lifecycle", "onCreate called - Activity born! 🎂")
}
```

**Real Example:**
```kotlin
class MainActivity : AppCompatActivity() {
    
    private lateinit var binding: ActivityMainBinding
    private lateinit var viewModel: MainViewModel
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // View Binding setup
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)
        
        // ViewModel
        viewModel = ViewModelProvider(this)[MainViewModel::class.java]
        
        // Setup UI
        setupUI()
        
        // Observe data
        observeData()
        
        Log.d("MainActivity", "onCreate: Activity created")
    }
    
    private fun setupUI() {
        binding.btnSubmit.setOnClickListener {
            val name = binding.etName.text.toString()
            viewModel.submitName(name)
        }
    }
    
    private fun observeData() {
        viewModel.userName.observe(this) { name ->
            binding.tvWelcome.text = "Welcome, $name!"
        }
    }
}
```

**Memory Trick:**
```
onCreate = জন্মদিনে যা করো
- নাম দাও (setContentView)
- পরিচয় করাও (findViewById)
- খেলনা দাও (initialize objects)
```

---

### 2️⃣ onStart() - Activity দৃশ্যমান হচ্ছে

**কখন call হয়:**
```
✅ onCreate() এর পরে
✅ onRestart() এর পরে
✅ Activity screen এ দেখা যাচ্ছে কিন্তু interact করা যাচ্ছে না
```

**কি করতে হয়:**
```kotlin
override fun onStart() {
    super.onStart()
    
    // Animation শুরু করতে পারো
    startAnimation()
    
    // UI update করতে পারো
    updateUI()
    
    // Broadcast receiver register করতে পারো
    registerReceiver(myReceiver, IntentFilter("ACTION"))
    
    Log.d("Lifecycle", "onStart called - User can see me! 👀")
}
```

**Real Example:**
```kotlin
override fun onStart() {
    super.onStart()
    
    // Analytics tracking
    analyticsTracker.trackScreenView("MainActivity")
    
    // Start animations
    binding.ivLogo.animate()
        .alpha(1f)
        .setDuration(500)
        .start()
    
    Log.d("MainActivity", "onStart: Activity visible")
}
```

---

### 3️⃣ onResume() - Activity সক্রিয়!

**কখন call হয়:**
```
✅ onStart() এর পরে
✅ onPause() থেকে ফিরে আসলে
✅ Activity fully interactive এখন!
✅ User এর সাথে interact করতে পারবে
```

**কি করতে হয়:**
```kotlin
override fun onResume() {
    super.onResume()
    
    // Camera start করো
    cameraManager.startCamera()
    
    // Location updates শুরু করো
    locationManager.requestLocationUpdates()
    
    // Music play করো
    mediaPlayer.start()
    
    // Sensor listening শুরু করো
    sensorManager.registerListener()
    
    Log.d("Lifecycle", "onResume called - Ready to interact! ⚡")
}
```

**Real Example:**
```kotlin
class CameraActivity : AppCompatActivity() {
    
    private lateinit var cameraManager: CameraManager
    
    override fun onResume() {
        super.onResume()
        
        // Camera permission check
        if (hasCameraPermission()) {
            cameraManager.startCamera()
            Log.d("CameraActivity", "Camera started")
        }
        
        // Screen always on যখন এই activity active
        window.addFlags(WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON)
    }
}
```

**Important Note:**
```
⚠️ onResume() এ heavy operations করো না!
⚠️ এখানে দেরি হলে UI lag করবে
⚠️ Quick operations only!
```

---

### 4️⃣ onPause() - Activity আংশিক বিরতি

**কখন call হয়:**
```
✅ Phone call আসলে
✅ Dialog দেখালে
✅ Another activity partially cover করলে
✅ User চলে যাচ্ছে (কিন্তু Activity এখনো দেখা যাচ্ছে)
✅ Multi-window mode এ অন্য window focus পেলে
```

**কি করতে হয়:**
```kotlin
override fun onPause() {
    super.onPause()
    
    // Camera stop করো
    cameraManager.stopCamera()
    
    // Location updates বন্ধ করো
    locationManager.removeLocationUpdates()
    
    // Music pause করো
    mediaPlayer.pause()
    
    // Animation pause করো
    animationDrawable.stop()
    
    // Important data save করো (quick!)
    saveUserInput()
    
    Log.d("Lifecycle", "onPause called - Going to background... 😴")
}
```

**Real Example:**
```kotlin
class VideoPlayerActivity : AppCompatActivity() {
    
    private lateinit var exoPlayer: ExoPlayer
    private var playbackPosition = 0L
    
    override fun onPause() {
        super.onPause()
        
        // Video pause করো
        exoPlayer.pause()
        
        // Current position save করো
        playbackPosition = exoPlayer.currentPosition
        
        // Screen always on flag remove করো
        window.clearFlags(WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON)
        
        Log.d("VideoPlayer", "onPause: Video paused at $playbackPosition")
    }
}
```

**Critical Rules:**
```
⚠️ onPause() MUST be FAST! (< 100ms)
⚠️ Heavy operations করো না এখানে
⚠️ Database operations এখানে না
⚠️ Network calls এখানে না
⚠️ Quick save operations only!
```

---

### 5️⃣ onStop() - Activity সম্পূর্ণ বিরতি

**কখন call হয়:**
```
✅ Activity সম্পূর্ণ invisible হলে
✅ Home button চাপলে
✅ Another activity fully cover করলে
✅ User navigate করে অন্য screen এ গেলে
```

**কি করতে হয়:**
```kotlin
override fun onStop() {
    super.onStop()
    
    // Database এ data save করো
    saveDataToDatabase()
    
    // Broadcast receiver unregister করো
    unregisterReceiver(myReceiver)
    
    // Network calls cancel করো
    cancelNetworkRequests()
    
    // Heavy cleanup operations
    performCleanup()
    
    Log.d("Lifecycle", "onStop called - Fully hidden! 🙈")
}
```

**Real Example:**
```kotlin
class FormActivity : AppCompatActivity() {
    
    private lateinit var database: AppDatabase
    
    override fun onStop() {
        super.onStop()
        
        // Form data save করো database এ
        val formData = FormData(
            name = binding.etName.text.toString(),
            email = binding.etEmail.text.toString(),
            phone = binding.etPhone.text.toString()
        )
        
        lifecycleScope.launch {
            database.formDao().insertFormData(formData)
            Log.d("FormActivity", "Form data saved")
        }
    }
}
```

**onPause vs onStop:**
```
┌───────────────┬──────────────┬──────────────┐
│               │  onPause()   │   onStop()   │
├───────────────┼──────────────┼──────────────┤
│ Visibility    │ Partially    │ Fully        │
│               │ visible      │ hidden       │
├───────────────┼──────────────┼──────────────┤
│ Speed         │ FAST!        │ Can be slow  │
│               │ < 100ms      │              │
├───────────────┼──────────────┼──────────────┤
│ Operations    │ Quick save   │ DB, Network  │
│               │ Pause media  │ Heavy ops    │
├───────────────┼──────────────┼──────────────┤
│ Example       │ Phone call   │ Home button  │
└───────────────┴──────────────┴──────────────┘
```

---

### 6️⃣ onRestart() - Activity পুনরায় শুরু

**কখন call হয়:**
```
✅ onStop() থেকে ফিরে আসার সময়
✅ User back button দিয়ে ফিরলে
✅ Recent apps থেকে select করলে
```

**কি করতে হয়:**
```kotlin
override fun onRestart() {
    super.onRestart()
    
    // Data refresh করো
    refreshData()
    
    // Analytics track করো
    trackRestart()
    
    Log.d("Lifecycle", "onRestart called - Coming back! 🔄")
    
    // পরে onStart() call হবে automatically
}
```

**Real Example:**
```kotlin
class NewsActivity : AppCompatActivity() {
    
    override fun onRestart() {
        super.onRestart()
        
        // News refresh করো (user ফিরে এসেছে)
        viewModel.refreshNews()
        
        // Notification clear করো
        notificationManager.cancelAll()
        
        Log.d("NewsActivity", "onRestart: Refreshing news")
    }
}
```

**Flow:**
```
onStop() → User চলে গেছে
    ↓
(User ফিরে এলো)
    ↓
onRestart() → Restart হচ্ছে
    ↓
onStart() → দেখা যাচ্ছে
    ↓
onResume() → Active!
```

---

### 7️⃣ onDestroy() - Activity এর মৃত্যু

**কখন call হয়:**
```
✅ finish() call করলে
✅ Back button চাপলে
✅ System memory recover করতে চাইলে
✅ Configuration change (screen rotate)
```

**কি করতে হয়:**
```kotlin
override fun onDestroy() {
    super.onDestroy()
    
    // Database connection close করো
    database.close()
    
    // MediaPlayer release করো
    mediaPlayer.release()
    
    // Coroutines cancel করো
    job.cancel()
    
    // Listeners remove করো
    removeAllListeners()
    
    Log.d("Lifecycle", "onDestroy called - Goodbye! 👋")
}
```

**Real Example:**
```kotlin
class MusicPlayerActivity : AppCompatActivity() {
    
    private var mediaPlayer: MediaPlayer? = null
    private var job: Job? = null
    
    override fun onDestroy() {
        super.onDestroy()
        
        // Media player release করো (memory free!)
        mediaPlayer?.release()
        mediaPlayer = null
        
        // Coroutines cancel করো
        job?.cancel()
        
        // Service unbind করো
        unbindService(serviceConnection)
        
        Log.d("MusicPlayer", "onDestroy: Resources cleaned up")
    }
}
```

**Important:**
```
⚠️ onDestroy() guaranteed না!
⚠️ System kill করলে call নাও হতে পারে
⚠️ Critical data save এখানে না, onPause/onStop এ করো
⚠️ Memory cleanup এর জন্য perfect জায়গা
```

---

## 🎯 Real-World Scenarios

### Scenario 1: Phone Call আসলো ☎️

```
Running Activity
       ↓
   onPause()   ← Call screen আসছে (আংশিক দেখা যাচ্ছে)
       ↓
   onStop()    ← Call screen পুরো দেখা যাচ্ছে
       ↓
(Call শেষ হলো)
       ↓
   onRestart() ← Activity ফিরে আসছে
       ↓
   onStart()   ← দেখা যাচ্ছে
       ↓
   onResume()  ← Active আবার!
```

**Code Example:**
```kotlin
class GameActivity : AppCompatActivity() {
    
    private var gameEngine: GameEngine? = null
    private var isPaused = false
    
    override fun onPause() {
        super.onPause()
        // Game pause করো (phone call এর জন্য)
        gameEngine?.pause()
        isPaused = true
        Log.d("GameActivity", "Game paused due to phone call")
    }
    
    override fun onResume() {
        super.onResume()
        // Game resume করো
        if (isPaused) {
            gameEngine?.resume()
            isPaused = false
            Log.d("GameActivity", "Game resumed")
        }
    }
}
```

---

### Scenario 2: Home Button চাপলো 🏠

```
Running Activity
       ↓
   onPause()   ← Launcher আসছে
       ↓
   onStop()    ← Activity completely hidden
       ↓
(Recent apps থেকে select করলো)
       ↓
   onRestart()
       ↓
   onStart()
       ↓
   onResume()  ← Active!
```

**Code Example:**
```kotlin
class ShoppingCartActivity : AppCompatActivity() {
    
    override fun onStop() {
        super.onStop()
        
        // Cart data save করো (user home button চাপলো)
        val cartItems = viewModel.getCartItems()
        saveCartToPreferences(cartItems)
        
        Log.d("ShoppingCart", "Cart saved: ${cartItems.size} items")
    }
    
    override fun onRestart() {
        super.onRestart()
        
        // Cart data restore করো
        val savedCart = loadCartFromPreferences()
        viewModel.restoreCart(savedCart)
        
        Log.d("ShoppingCart", "Cart restored: ${savedCart.size} items")
    }
}
```

---

### Scenario 3: Screen Rotate করলো 🔄

```
Running Activity
       ↓
   onPause()
       ↓
   onStop()
       ↓
   onDestroy()  ← Activity destroyed (config change!)
       ↓
   onCreate()   ← নতুন Activity তৈরি হলো (landscape/portrait)
       ↓
   onStart()
       ↓
   onResume()
```

**Code Example:**
```kotlin
class FormActivity : AppCompatActivity() {
    
    private var userInput = ""
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_form)
        
        // Saved state restore করো
        savedInstanceState?.let {
            userInput = it.getString("USER_INPUT", "")
            binding.etInput.setText(userInput)
            Log.d("FormActivity", "Restored: $userInput")
        }
    }
    
    override fun onSaveInstanceState(outState: Bundle) {
        super.onSaveInstanceState(outState)
        
        // Data save করো rotate এর আগে
        userInput = binding.etInput.text.toString()
        outState.putString("USER_INPUT", userInput)
        
        Log.d("FormActivity", "Saving: $userInput")
    }
}
```

---

### Scenario 4: Back Button চাপলো ⬅️

```
Running Activity
       ↓
   onPause()
       ↓
   onStop()
       ↓
   onDestroy()  ← Activity finished!
       ↓
   (Dead - কবরস্থানে 💀)
```

**Code Example:**
```kotlin
class DetailActivity : AppCompatActivity() {
    
    override fun onBackPressed() {
        // Data save করে নাও
        val result = Intent().apply {
            putExtra("EDITED_DATA", editedData)
        }
        setResult(RESULT_OK, result)
        
        // এখন back যাবে
        super.onBackPressed()
    }
    
    override fun onDestroy() {
        super.onDestroy()
        
        // Cleanup
        viewModel.clearData()
        
        Log.d("DetailActivity", "Activity destroyed")
    }
}
```

---

## 💾 Data Save করার Strategy

### তিনটা Level:

#### 1. Temporary UI State (onSaveInstanceState)
```kotlin
// Rotate এর সময় data save
override fun onSaveInstanceState(outState: Bundle) {
    super.onSaveInstanceState(outState)
    
    outState.putString("SCROLL_POSITION", scrollPosition.toString())
    outState.putInt("SELECTED_TAB", selectedTab)
    outState.putString("SEARCH_QUERY", searchQuery)
}

// Restore করো onCreate এ
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    
    savedInstanceState?.let {
        scrollPosition = it.getString("SCROLL_POSITION")?.toInt() ?: 0
        selectedTab = it.getInt("SELECTED_TAB", 0)
        searchQuery = it.getString("SEARCH_QUERY", "")
    }
}
```

**কখন ব্যবহার করবে:**
- Screen rotate
- Temporary minimize
- UI state only (scroll position, selected tab)

---

#### 2. Session Data (SharedPreferences/DataStore)
```kotlin
override fun onPause() {
    super.onPause()
    
    // Quick save to preferences
    getPreferences(MODE_PRIVATE).edit {
        putString("LAST_SEARCH", searchQuery)
        putBoolean("DARK_MODE", isDarkMode)
        putLong("LAST_VISIT", System.currentTimeMillis())
    }
}
```

**কখন ব্যবহার করবে:**
- User preferences
- Settings
- Last state
- Quick saves

---

#### 3. Persistent Data (Room Database)
```kotlin
override fun onStop() {
    super.onStop()
    
    // Save to database
    lifecycleScope.launch {
        val draftPost = DraftPost(
            title = binding.etTitle.text.toString(),
            content = binding.etContent.text.toString(),
            timestamp = System.currentTimeMillis()
        )
        database.draftDao().insertDraft(draftPost)
    }
}
```

**কখন ব্যবহার করবে:**
- Important user data
- Drafts
- Offline data
- Complex objects

---

## 📊 Lifecycle Summary Table

| Method | Called When | Visible? | Interactive? | What to Do |
|--------|-------------|----------|--------------|------------|
| **onCreate()** | First time created | ❌ NO | ❌ NO | Initialize UI, ViewModel |
| **onStart()** | About to become visible | ✅ YES | ❌ NO | Animations, UI updates |
| **onResume()** | Ready for interaction | ✅ YES | ✅ YES | Start camera, sensors |
| **onPause()** | Losing focus | ✅ YES | ❌ NO | Pause media, save data |
| **onStop()** | No longer visible | ❌ NO | ❌ NO | Save to DB, cleanup |
| **onRestart()** | Coming back from stopped | ❌ NO | ❌ NO | Refresh data |
| **onDestroy()** | Being destroyed | ❌ NO | ❌ NO | Release resources |

---

## 🎯 Common Mistakes & Solutions

### ❌ Mistake 1: onCreate এ Heavy Operations
```kotlin
// ❌ WRONG - UI freeze করবে!
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)
    
    // Heavy operation - 5 seconds লাগছে!
    loadLargeDataset()  // 😱
}

// ✅ CORRECT - Background thread এ করো
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)
    
    lifecycleScope.launch {
        withContext(Dispatchers.IO) {
            loadLargeDataset()  // Background এ ✅
        }
    }
}
```

---

### ❌ Mistake 2: onPause এ Heavy Operations
```kotlin
// ❌ WRONG - App freeze করবে!
override fun onPause() {
    super.onPause()
    
    // Database এ 1000 items save - slow!
    saveAllItemsToDatabase()  // 😱
}

// ✅ CORRECT - onStop এ করো
override fun onPause() {
    super.onPause()
    // Quick operations only
    mediaPlayer.pause()
}

override fun onStop() {
    super.onStop()
    // Heavy operations এখানে OK
    lifecycleScope.launch {
        saveAllItemsToDatabase()  // ✅
    }
}
```

---

### ❌ Mistake 3: Memory Leaks
```kotlin
// ❌ WRONG - Memory leak!
class BadActivity : AppCompatActivity() {
    
    private val handler = Handler()  // Static reference!
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        handler.postDelayed({
            // Activity destroyed হলেও এটা চলবে! 😱
            updateUI()
        }, 10000)
    }
}

// ✅ CORRECT - Cleanup করো
class GoodActivity : AppCompatActivity() {
    
    private val handler = Handler(Looper.getMainLooper())
    private val updateRunnable = Runnable { updateUI() }
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        handler.postDelayed(updateRunnable, 10000)
    }
    
    override fun onDestroy() {
        super.onDestroy()
        handler.removeCallbacks(updateRunnable)  // ✅ Cleanup!
    }
}
```

---

### ❌ Mistake 4: Data Save না করা
```kotlin
// ❌ WRONG - Screen rotate এ data lost!
class BadFormActivity : AppCompatActivity() {
    
    private var formData = ""
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // formData restore করছি না!
    }
}

// ✅ CORRECT - Save & Restore
class GoodFormActivity : AppCompatActivity() {
    
    private var formData = ""
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // Restore
        formData = savedInstanceState?.getString("FORM_DATA") ?: ""
    }
    
    override fun onSaveInstanceState(outState: Bundle) {
        super.onSaveInstanceState(outState)
        
        // Save
        outState.putString("FORM_DATA", formData)
    }
}
```

---

## 🎯 Best Practices Checklist

```
✅ onCreate() এ UI initialize করো
✅ onStart() এ animations শুরু করো
✅ onResume() এ sensors/camera start করো
✅ onPause() MUST be fast (< 100ms)
✅ onPause() এ media pause করো
✅ onStop() এ database save করো
✅ onDestroy() এ resources release করো
✅ Screen rotate handle করো (onSaveInstanceState)
✅ Memory leaks prevent করো
✅ Background threads use করো heavy operations এর জন্য
```

---

## 🧪 Testing Lifecycle

### Manually Test করো:

```
1. App Launch → onCreate, onStart, onResume check করো

2. Home Button চাপো → onPause, onStop check করো

3. Recent Apps থেকে select করো → onRestart, onStart, onResume check করো

4. Phone Call receive করো → onPause check করো

5. Screen Rotate করো → 
   - Old: onPause, onStop, onDestroy
   - New: onCreate, onStart, onResume
   - Data saved/restored কিনা check করো

6. Back Button চাপো → onPause, onStop, onDestroy check করো

7. Another Activity open করো → onPause, onStop check করো
```

---

## 💡 Memory Tricks

### মনে রাখার সহজ উপায়:

```
📱 Activity Lifecycle = মানুষের একদিন

onCreate()   = ঘুম থেকে উঠা (জন্ম)
onStart()    = চোখ খোলা (দেখা যাচ্ছে)
onResume()   = কাজ শুরু (সক্রিয়)
onPause()    = বিরতি নিলো (pause)
onStop()     = ঘুমিয়ে গেলো (invisible)
onRestart()  = আবার উঠলো (restart)
onDestroy()  = দিন শেষ (মৃত্যু)
```

### Acronym: "**CSR-PSD**"
```
C - Create
S - Start
R - Resume
---
P - Pause
S - Stop
D - Destroy
```

---

## 🎓 Interview Questions & Answers

### Q1: Activity Lifecycle কি?
```
Answer: 
Activity Lifecycle হলো একটা Activity এর বিভিন্ন states এর মধ্যে 
transition। এটা 7টা callback method দিয়ে manage হয়:
onCreate, onStart, onResume, onPause, onStop, onRestart, onDestroy।
```

### Q2: onPause vs onStop এর পার্থক্য?
```
Answer:
┌────────────┬──────────────┬──────────────┐
│            │  onPause()   │   onStop()   │
├────────────┼──────────────┼──────────────┤
│ Visibility │ Partially    │ Fully        │
│            │ visible      │ hidden       │
├────────────┼──────────────┼──────────────┤
│ Speed      │ MUST be fast │ Can be slow  │
│            │ (< 100ms)    │              │
├────────────┼──────────────┼──────────────┤
│ Example    │ Phone call,  │ Home button, │
│            │ Dialog       │ New activity │
└────────────┴──────────────┴──────────────┘
```

### Q3: Screen rotate করলে কি হয়?
```
Answer:
Activity destroy হয়ে আবার create হয়:
1. onPause() → onStop() → onDestroy()
2. onCreate() → onStart() → onResume()

Data save করতে হয় onSaveInstanceState() এ
এবং restore করতে হয় onCreate() এ।
```

### Q4: onDestroy() guaranteed কি?
```
Answer:
না! System memory recover করার জন্য kill করলে 
onDestroy() call নাও হতে পারে।

তাই critical data save করতে হয় onPause() বা 
onStop() এ, onDestroy() এ না।
```

---

**Created by: Claude for Hasibuzzaman Chowdhury**  
**Date: December 30, 2025**  
**মনে রাখো: Lifecycle master করলে তুমি 50% Android জিতে গেছো! 🚀**
