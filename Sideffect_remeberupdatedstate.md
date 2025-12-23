# rememberUpdatedState - Complete Professional Tutorial 🎯

## সূচিপত্র

1. [Professional Context: কেন এবং কোথায়](#1-professional-context)
2. [মূল সমস্যা: Stale Closure](#2-মূল-সমস্যা-stale-closure)
3. [rememberUpdatedState কীভাবে কাজ করে](#3-কীভাবে-কাজ-করে)
4. [Professional Use Cases](#4-professional-use-cases)
5. [Pattern & Best Practices](#5-pattern--best-practices)
6. [Quick Reference](#6-quick-reference)

---

## 1. Professional Context

### প্রফেশনাল ডেভেলপাররা কেন ব্যবহার করেন?

```
প্রফেশনাল লেভেলে অ্যান্ড্রয়েড ডেভেলপমেন্টে rememberUpdatedState মূলত সেখানে ব্যবহার করা হয় যেখানে কোনো Side Effect
(যেমন: LaunchedEffect, DisposableEffect) এর ভেতর কোনো ভ্যালু ব্যবহার করা হচ্ছে, কিন্তু ভ্যালু চেঞ্জ হওয়ার কারণে আমরা
সেই ইফেক্টটিকে বারবার রিস্টার্ট (Restart) করতে চাই না।
rememberUpdatedState = Side Effect এর ভেতর Latest Value নিশ্চিত করা
                      কিন্তু Effect Restart করা ছাড়াই!
```

#### তিনটা মূল কারণ:

### 🚀 1. Performance (অপ্রয়োজনীয় Restart বন্ধ)

```kotlin
// ❌ Without rememberUpdatedState
LaunchedEffect(callback) {  // callback change → restart!
    delay(60000)
    callback()
}

// ✅ With rememberUpdatedState
val current by rememberUpdatedState(callback)
LaunchedEffect(Unit) {  // No restart!
    delay(60000)
    current()  // Latest callback
}
```

**Performance Impact:**
```
Without: 10 recompositions → 10 restarts → Inefficient
With: 10 recompositions → 0 restarts → Efficient! 🎯
```

---

### ✅ 2. Correctness (সবসময় Latest Value)

```kotlin
@Composable
fun TrackScreen(screenName: String) {
    // ❌ Stale value
    DisposableEffect(Unit) {
        onDispose {
            log(screenName)  // May be old!
        }
    }
    
    // ✅ Latest value
    val current by rememberUpdatedState(screenName)
    DisposableEffect(Unit) {
        onDispose {
            log(current)  // Always latest!
        }
    }
}
```

---

### 🔒 3. Stability (Long-running Tasks)

```kotlin
// Timer, Network listeners, Sensors
// চলতে থাকবে, কিন্তু latest callback ব্যবহার করবে

LaunchedEffect(Unit) {  // Stable, won't restart
    val currentCallback by rememberUpdatedState(callback)
    while (true) {
        delay(5000)
        currentCallback()  // Latest callback every 5s
    }
}
```

---

## 2. মূল সমস্যা: Stale Closure

### 🍕 Real Life Analogy

```
Foodpanda থেকে Pizza order:
- Address: "Mirpur 10"
- Delivery guy রওনা দিলো

5 মিনিট পর:
- আপনি address change করলেন: "Dhanmondi 15"

কিন্তু:
- Delivery guy এখনো Mirpur 10 এর দিকে যাচ্ছে!
- তার কাছে পুরানো address আছে

❌ Pizza পৌঁছাবে ভুল জায়গায়!
```

**এটাই Stale Closure Problem!** 🐛

---

### Code Example: The Problem

```kotlin
@Composable
fun TimeoutScreen(onTimeout: () -> Unit) {
    
    // ❌ Problem: onTimeout captured at start
    LaunchedEffect(Unit) {
        delay(5000)
        onTimeout()  // Stale callback!
    }
}

@Composable
fun ParentScreen() {
    var message by remember { mutableStateOf("Initial") }
    
    TimeoutScreen(
        onTimeout = { 
            println("Message: $message")
        }
    )
    
    // 2 seconds পর change
    LaunchedEffect(Unit) {
        delay(2000)
        message = "Updated"
    }
}
```

**Output:**
```
After 2s: message = "Updated"
After 5s: "Message: Initial"  ← ❌ Stale!
```

---

### ✅ Solution: rememberUpdatedState

```kotlin
@Composable
fun TimeoutScreen(onTimeout: () -> Unit) {
    
    // ✅ Always get latest
    val currentOnTimeout by rememberUpdatedState(onTimeout)
    
    LaunchedEffect(Unit) {
        delay(5000)
        currentOnTimeout()  // Latest callback!
    }
}
```

**Output:**
```
After 2s: message = "Updated"
After 5s: "Message: Updated"  ← ✅ Latest!
```

---

## 3. কীভাবে কাজ করে

### Internal Mechanism

```kotlin
// rememberUpdatedState simplified implementation
@Composable
fun <T> rememberUpdatedState(newValue: T): State<T> {
    val state = remember { mutableStateOf(newValue) }
    
    // প্রতি recomposition এ update
    state.value = newValue
    
    return state
}
```

### Execution Flow

```
Initial Composition:
1. Creates State object
2. Sets value = initial

Recomposition (value changed):
3. Same State object (remembered)
4. Updates State.value = new value

LaunchedEffect accesses:
5. Reads State.value → Gets latest! ✅
```

---

## 4. Professional Use Cases

### Use Case 1: 📊 Analytics & Logging (সবচেয়ে Common!)

#### Real Scenario:
```
Screen এর analytics track করছেন।
Screen থেকে leave করার সময় log পাঠাতে হবে।
কিন্তু screen name বা data dynamic।
```

#### ❌ Problem Without rememberUpdatedState

```kotlin
@Composable
fun ProductDetailsScreen(productId: String) {
    var viewDuration by remember { mutableStateOf(0L) }
    
    // ❌ Logs stale productId
    DisposableEffect(Unit) {
        val startTime = System.currentTimeMillis()
        
        onDispose {
            viewDuration = System.currentTimeMillis() - startTime
            
            // Problem: productId may be old!
            Analytics.logEvent("product_viewed", mapOf(
                "product_id" to productId,
                "duration" to viewDuration
            ))
        }
    }
    
    // User scrolls, productId changes
    ProductContent(productId)
}
```

**Scenario:**
```
User opens: productId = "123"
User scrolls quickly to: productId = "456"
User leaves screen

Analytics logs:
product_id: "123"  ← ❌ Wrong! Should be "456"
```

---

#### ✅ Solution with rememberUpdatedState

```kotlin
@Composable
fun ProductDetailsScreen(productId: String) {
    var viewDuration by remember { mutableStateOf(0L) }
    
    // ✅ Always logs latest productId
    val currentProductId by rememberUpdatedState(productId)
    val currentDuration by rememberUpdatedState(viewDuration)
    
    DisposableEffect(Unit) {
        val startTime = System.currentTimeMillis()
        
        onDispose {
            viewDuration = System.currentTimeMillis() - startTime
            
            // ✅ Latest values!
            Analytics.logEvent("product_viewed", mapOf(
                "product_id" to currentProductId,
                "duration" to currentDuration
            ))
        }
    }
    
    ProductContent(productId)
}
```

**Result:**
```
User opens: productId = "123"
User scrolls to: productId = "456"
User leaves screen

Analytics logs:
product_id: "456"  ← ✅ Correct!
duration: 5000ms
```

---

### Use Case 2: 🔐 OTP Timer (Time-sensitive Actions)

#### Real Scenario:
```
OTP screen এ 60 seconds timer।
Timer চলাকালীন user phone number change করতে পারে।
কিন্তু timer restart করা চাই না।
```

#### Implementation

```kotlin
@Composable
fun OtpScreen() {
    var phoneNumber by remember { mutableStateOf("+8801711111111") }
    var otpSent by remember { mutableStateOf(false) }
    var canResend by remember { mutableStateOf(false) }
    
    // ✅ Latest phone number ব্যবহার হবে
    val currentPhone by rememberUpdatedState(phoneNumber)
    
    LaunchedEffect(otpSent) {
        if (otpSent) {
            canResend = false
            
            // 60 seconds timer
            delay(60000L)
            
            canResend = true
            println("✅ Can resend to: $currentPhone")
        }
    }
    
    Column {
        TextField(
            value = phoneNumber,
            onValueChange = { phoneNumber = it },
            label = { Text("Phone Number") }
        )
        
        Button(
            onClick = { 
                otpSent = true
                sendOTP(phoneNumber)
            },
            enabled = !otpSent || canResend
        ) {
            Text(if (canResend) "Resend OTP" else "Send OTP")
        }
        
        if (otpSent && !canResend) {
            Text("Wait 60 seconds...")
        }
    }
}
```

**Timeline:**
```
0s:  phoneNumber = "+8801711111111"
     User clicks "Send OTP"
     Timer starts

15s: User realizes wrong number
     Changes to: "+8801722222222"
     Timer continues (no restart!)

60s: Timer completes
     "Can resend to: +8801722222222" ← ✅ Latest number!
```

**Without rememberUpdatedState:**
```
60s: "Can resend to: +8801711111111" ← ❌ Old number!
```

---

### Use Case 3: 🎵 Music Player Auto-Next

#### Real Scenario:
```
Music player এ song চলছে।
3 minutes পর auto-next।
কিন্তু user মাঝপথে next song change করতে পারে।
```

#### Implementation

```kotlin
data class Song(val id: String, val name: String, val duration: Long)

@Composable
fun MusicPlayer() {
    var currentSong by remember { mutableStateOf(Song("1", "Song 1", 180000)) }
    var nextSong by remember { mutableStateOf(Song("2", "Song 2", 200000)) }
    var isPlaying by remember { mutableStateOf(false) }
    
    // ✅ Latest next song play হবে
    val currentNextSong by rememberUpdatedState(nextSong)
    
    LaunchedEffect(currentSong, isPlaying) {
        if (isPlaying) {
            println("▶️ Playing: ${currentSong.name}")
            
            // Song duration পর auto-next
            delay(currentSong.duration)
            
            println("⏭️ Auto-playing next: ${currentNextSong.name}")
            currentSong = currentNextSong
        }
    }
    
    Column {
        Text("🎵 Now: ${currentSong.name}")
        Text("⏭️ Next: ${nextSong.name}")
        
        Button(onClick = { isPlaying = !isPlaying }) {
            Text(if (isPlaying) "⏸️ Pause" else "▶️ Play")
        }
        
        Button(onClick = { 
            nextSong = Song("3", "Song 3", 150000)
        }) {
            Text("Change Next")
        }
    }
}
```

**Scenario:**
```
0s:   Playing "Song 1", Next = "Song 2"
      Timer: 180s

60s:  User changes Next to "Song 3"
      Timer continues (120s remaining)

180s: Auto-next triggers
      Plays: "Song 3" ← ✅ Latest!
```

---

### Use Case 4: 📍 Location Tracker

#### Real Scenario:
```
Location sensor থেকে continuous updates।
Location পাওয়ার পর callback function run হবে।
Callback change হতে পারে, কিন্তু sensor restart চাই না।
```

#### Implementation

```kotlin
@Composable
fun LocationTracker(
    updateInterval: Long = 5000L,
    onLocationUpdate: (Location) -> Unit
) {
    // ✅ Latest callback ব্যবহার হবে
    val currentCallback by rememberUpdatedState(onLocationUpdate)
    
    DisposableEffect(Unit) {
        val locationListener = object : LocationListener {
            override fun onLocationChanged(location: Location) {
                // ✅ সবসময় latest callback!
                currentCallback(location)
            }
        }
        
        locationManager.requestLocationUpdates(
            LocationManager.GPS_PROVIDER,
            updateInterval,
            0f,
            locationListener
        )
        
        println("📡 Location tracking started")
        
        onDispose {
            locationManager.removeUpdates(locationListener)
            println("📡 Location tracking stopped")
        }
    }
}

@Composable
fun MapScreen() {
    var serverUrl by remember { mutableStateOf("https://api-v1.com") }
    var userLocation by remember { mutableStateOf<Location?>(null) }
    
    LocationTracker(
        onLocationUpdate = { location ->
            userLocation = location
            
            // ✅ Latest server URL এ send হবে!
            sendToServer(serverUrl, location)
            println("📍 Sent to: $serverUrl")
        }
    )
    
    Column {
        Text("📍 Location: ${userLocation?.latitude}, ${userLocation?.longitude}")
        Text("🌐 Server: $serverUrl")
        
        Button(onClick = { serverUrl = "https://api-v2.com" }) {
            Text("Switch Server")
        }
    }
}
```

**Timeline:**
```
0s:  Tracking starts, serverUrl = "api-v1.com"

5s:  Location update
     Sent to: api-v1.com ✅

8s:  User switches to: "api-v2.com"
     Tracking continues (no restart!)

10s: Location update
     Sent to: api-v2.com ← ✅ Latest server!
```

---

### Use Case 5: 🔵 Bluetooth Device Listener

#### Real Scenario:
```
Bluetooth device থেকে data receive করছেন।
Data আসার পর handler function call হবে।
Handler change হতে পারে, কিন্তু connection restart চাই না।
```

#### Implementation

```kotlin
@Composable
fun BluetoothDataReceiver(
    deviceAddress: String,
    onDataReceived: (ByteArray) -> Unit
) {
    // ✅ Latest handler ব্যবহার হবে
    val currentHandler by rememberUpdatedState(onDataReceived)
    
    DisposableEffect(deviceAddress) {  // Restart only if device changes
        val callback = object : BluetoothCallback {
            override fun onDataReceived(data: ByteArray) {
                // ✅ Always latest handler!
                currentHandler(data)
            }
        }
        
        bluetoothManager.connect(deviceAddress, callback)
        println("🔵 Connected to: $deviceAddress")
        
        onDispose {
            bluetoothManager.disconnect(deviceAddress)
            println("🔵 Disconnected from: $deviceAddress")
        }
    }
}

@Composable
fun SensorScreen() {
    var sensorData by remember { mutableStateOf("") }
    var logLevel by remember { mutableStateOf("INFO") }
    
    BluetoothDataReceiver(
        deviceAddress = "00:11:22:33:44:55",
        onDataReceived = { data ->
            sensorData = data.decodeToString()
            
            // ✅ Latest log level ব্যবহার হবে!
            when (logLevel) {
                "DEBUG" -> Log.d("Sensor", sensorData)
                "INFO" -> Log.i("Sensor", sensorData)
                "ERROR" -> Log.e("Sensor", sensorData)
            }
        }
    )
    
    Column {
        Text("📊 Data: $sensorData")
        
        Row {
            Button(onClick = { logLevel = "DEBUG" }) { Text("Debug") }
            Button(onClick = { logLevel = "INFO" }) { Text("Info") }
            Button(onClick = { logLevel = "ERROR" }) { Text("Error") }
        }
    }
}
```

---

### Use Case 6: 🛒 Shopping Cart Auto-Save

#### Real Scenario:
```
Shopping cart auto-save every 30 seconds।
Cart items change হতে পারে frequent।
কিন্তু auto-save timer restart চাই না।
```

#### Implementation

```kotlin
data class CartItem(val id: String, val name: String, val price: Double)

@Composable
fun ShoppingCart() {
    var items by remember { mutableStateOf(listOf<CartItem>()) }
    var lastSaved by remember { mutableStateOf<Long?>(null) }
    var isSaving by remember { mutableStateOf(false) }
    
    // ✅ Latest cart save হবে
    val currentItems by rememberUpdatedState(items)
    
    // Auto-save every 30 seconds
    LaunchedEffect(Unit) {
        while (true) {
            delay(30000)
            
            isSaving = true
            println("💾 Auto-saving cart...")
            
            // ✅ Latest items!
            saveCartToServer(currentItems)
            
            lastSaved = System.currentTimeMillis()
            isSaving = false
            
            println("✅ Saved ${currentItems.size} items")
        }
    }
    
    Column {
        Text("🛒 Cart: ${items.size} items")
        
        if (isSaving) {
            Text("💾 Saving...")
        }
        
        lastSaved?.let {
            Text("Last saved: ${formatTime(it)}")
        }
        
        items.forEach { item ->
            Row {
                Text("${item.name} - ৳${item.price}")
                Button(onClick = { 
                    items = items - item 
                }) {
                    Text("Remove")
                }
            }
        }
        
        Button(onClick = { 
            items = items + CartItem(
                id = UUID.randomUUID().toString(),
                name = "Product ${items.size + 1}",
                price = 100.0
            )
        }) {
            Text("Add Item")
        }
    }
}
```

**Timeline:**
```
0s:   Cart: []
      Auto-save timer starts

5s:   User adds Item 1
      Cart: [Item 1]

15s:  User adds Item 2, Item 3
      Cart: [Item 1, Item 2, Item 3]

30s:  Auto-save triggers
      Saves: [Item 1, Item 2, Item 3] ← ✅ Latest!

35s:  User removes Item 2
      Cart: [Item 1, Item 3]

60s:  Auto-save triggers
      Saves: [Item 1, Item 3] ← ✅ Latest!
```

---

### Use Case 7: 🎮 Game Auto-Save

```kotlin
data class GameState(
    val level: Int,
    val score: Int,
    val playerName: String,
    val achievements: List<String>
)

@Composable
fun GameScreen() {
    var gameState by remember { 
        mutableStateOf(GameState(1, 0, "Player1", emptyList())) 
    }
    
    // ✅ Latest game state save হবে
    val currentState by rememberUpdatedState(gameState)
    
    // Auto-save every minute
    LaunchedEffect(Unit) {
        while (true) {
            delay(60000)
            
            println("💾 Auto-saving game...")
            saveGameToCloud(currentState)
            println("✅ Saved: Level ${currentState.level}, Score ${currentState.score}")
        }
    }
    
    Column {
        Text("🎮 Player: ${gameState.playerName}")
        Text("📊 Level: ${gameState.level}")
        Text("⭐ Score: ${gameState.score}")
        
        Button(onClick = { 
            gameState = gameState.copy(
                level = gameState.level + 1,
                score = gameState.score + 1000
            )
        }) {
            Text("Level Up!")
        }
        
        Button(onClick = { 
            gameState = gameState.copy(
                achievements = gameState.achievements + "New Achievement!"
            )
        }) {
            Text("Unlock Achievement")
        }
    }
}
```

---

## 5. Pattern & Best Practices

### 🎯 Common Pattern

```kotlin
@Composable
fun MyComponent(
    value: T,
    onEvent: (T) -> Unit
) {
    // ✅ Pattern: Capture latest
    val currentValue by rememberUpdatedState(value)
    val currentOnEvent by rememberUpdatedState(onEvent)
    
    LaunchedEffect(key) {  // Stable key
        // Long-running operation
        delay(...)
        
        // Use latest values
        currentOnEvent(currentValue)
    }
}
```

---

### ✅ Best Practices

#### 1. **Use with Side Effects Only**

```kotlin
// ✅ Good - In LaunchedEffect
val current by rememberUpdatedState(callback)
LaunchedEffect(Unit) {
    delay(5000)
    current()
}

// ❌ Bad - In regular composition
val current by rememberUpdatedState(value)
Text("Value: $current")  // Just use value directly!
```

---

#### 2. **Stable Keys for Effect**

```kotlin
// ✅ Good - Unit or stable key
LaunchedEffect(Unit) {
    val current by rememberUpdatedState(callback)
    // ...
}

// ⚠️ Be careful - restart on every change
LaunchedEffect(callback) {
    // Restarts when callback changes
}
```

---

#### 3. **Multiple Values**

```kotlin
// ✅ Capture all needed values
val currentX by rememberUpdatedState(x)
val currentY by rememberUpdatedState(y)
val currentZ by rememberUpdatedState(z)

LaunchedEffect(Unit) {
    delay(5000)
    process(currentX, currentY, currentZ)
}
```

---

#### 4. **Don't Overuse**

```kotlin
// ❌ Unnecessary - short delay
LaunchedEffect(Unit) {
    delay(100)  // Too short for stale closure
    doSomething(value)
}

// ❌ Unnecessary - stable key already
LaunchedEffect(userId) {  // Restarts when userId changes
    fetchUser(userId)  // No stale closure issue
}
```

---

### 📊 Decision Framework

```
Need value in LaunchedEffect/DisposableEffect?
    │
    ├─ Operation < 500ms?
    │   └─ No rememberUpdatedState needed
    │
    ├─ Effect key already includes value?
    │   └─ No rememberUpdatedState needed
    │
    ├─ Long-running operation?
    │   └─ ✅ Use rememberUpdatedState
    │
    └─ Callback might change?
        └─ ✅ Use rememberUpdatedState
```

---

## 6. Quick Reference

### Syntax

```kotlin
val currentValue by rememberUpdatedState(value)
```

---

### Use Cases Summary

| Use Case | কেন দরকার | Example |
|----------|-----------|---------|
| **Analytics** | Screen leave এ latest data log | Product view tracking |
| **Timers** | Timer চলাকালীন latest callback | OTP resend timer |
| **Sensors** | Continuous updates, latest handler | Location, Bluetooth |
| **Auto-Save** | Periodic save, latest state | Cart, Game, Draft |
| **Music/Video** | Playback events, latest callback | Auto-next, Auto-pause |

---

### Common Mistakes

#### ❌ Mistake 1: Using in Regular Composition

```kotlin
// ❌ Wrong
@Composable
fun Display(value: String) {
    val current by rememberUpdatedState(value)
    Text(current)  // Unnecessary!
}

// ✅ Correct
@Composable
fun Display(value: String) {
    Text(value)  // Direct use
}
```

---

#### ❌ Mistake 2: Not Using When Needed

```kotlin
// ❌ Wrong - Stale closure
LaunchedEffect(Unit) {
    delay(60000)
    callback()  // May be stale!
}

// ✅ Correct
val current by rememberUpdatedState(callback)
LaunchedEffect(Unit) {
    delay(60000)
    current()  // Always latest!
}
```

---

#### ❌ Mistake 3: Using with Unstable Key

```kotlin
// ⚠️ Be careful
LaunchedEffect(callback) {  // Restarts every change
    val current by rememberUpdatedState(callback)
    delay(5000)
    current()
}

// ✅ Better
val current by rememberUpdatedState(callback)
LaunchedEffect(Unit) {  // Stable, no restart
    delay(5000)
    current()
}
```

---

### When to Use

```kotlin
✅ Use rememberUpdatedState when:
   1. Long-running operation (> 1 second)
   2. Callbacks in LaunchedEffect/DisposableEffect
   3. Value might change during operation
   4. Don't want to restart effect

❌ Don't use when:
   1. Short operation (< 500ms)
   2. Effect key already includes value
   3. Regular composition (not in effect)
   4. Static/stable values
```

---

### Template Code

```kotlin
// Basic Template
@Composable
fun MyComponent(
    param: T,
    onEvent: (R) -> Unit
) {
    val currentParam by rememberUpdatedState(param)
    val currentOnEvent by rememberUpdatedState(onEvent)
    
    LaunchedEffect(Unit) {
        // Long operation
        val result = process(currentParam)
        currentOnEvent(result)
    }
}

// DisposableEffect Template
@Composable
fun ResourceManager(
    config: Config,
    onUpdate: (Data) -> Unit
) {
    val currentConfig by rememberUpdatedState(config)
    val currentOnUpdate by rememberUpdatedState(onUpdate)
    
    DisposableEffect(Unit) {
        val resource = acquire()
        
        resource.setListener { data ->
            currentOnUpdate(data)
        }
        
        onDispose {
            resource.release()
        }
    }
}
```

---

## 📝 Summary

### মূল Concept

```
Problem: LaunchedEffect এ value capture হয়ে যায় (stale)
Solution: rememberUpdatedState সবসময় latest value দেয়

Process:
1. State object তৈরি করে
2. প্রতি recomposition এ update করে
3. Effect latest value access করতে পারে
```

---

### Professional Benefits

```
🚀 Performance: Unnecessary restarts বন্ধ
✅ Correctness: Latest value guaranteed
🔒 Stability: Long-running tasks uninterrupted
```

---

### Most Common Use Cases

```
1. 📊 Analytics/Logging - Screen tracking
2. 🔐 Timers - OTP, Countdown
3. 📍 Sensors - Location, Bluetooth
4. 💾 Auto-Save - Cart, Game, Draft
5. 🎵 Media - Auto-next, Auto-pause
```

---

### Remember

```kotlin
Long delay + Value might change = rememberUpdatedState! 🎯

Pattern:
val current by rememberUpdatedState(value)
LaunchedEffect(Unit) {
    delay(...)
    use(current)  // Latest!
}
```

---

## 🎓 Final Tips

1. **Start simple**: ছোট example দিয়ে practice করুন
2. **Check logs**: Console output দেখে verify করুন
3. **Test scenarios**: Value change করে test করুন
4. **Use when needed**: Overuse করবেন না
5. **Follow patterns**: Template code follow করুন

---

**Happy Coding!** 🚀

**মনে রাখুন:**
```
LaunchedEffect + Long delay + Changing values
= rememberUpdatedState is your friend! 🎯
```
