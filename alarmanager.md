# Android AlarmManager - সম্পূর্ণ Tutorial (Bangla)

> **Exact time এ task execute করার জন্য Android এর native solution**

---

## 📚 Table of Contents
1. [AlarmManager কি?](#1-alarmmanager-কি)
2. [কেন AlarmManager ব্যবহার করবেন?](#2-কেন-alarmmanager-ব্যবহার-করবেন)
3. [AlarmManager vs WorkManager](#3-alarmmanager-vs-workmanager)
4. [Types of Alarms](#4-types-of-alarms)
5. [Simple Steps - কিভাবে কাজ করে](#5-simple-steps---কিভাবে-কাজ-করে-step-by-step)
6. [Complete Working Example (Minimal)](#6-complete-working-example-minimal)
7. [Real-world Example 1: Reminder App](#7-real-world-example-1-reminder-app)
8. [Real-world Example 2: Medicine Reminder](#8-real-world-example-2-medicine-reminder)
9. [Best Practices](#9-best-practices)
10. [Interview Questions](#10-interview-questions)

---

## 1. AlarmManager কি?

### Definition:

**AlarmManager** হলো Android system service যা **নির্দিষ্ট সময়ে** বা **নির্দিষ্ট interval এ** task execute করার জন্য ব্যবহৃত হয়।

**সহজ বাংলায়:**  
ঠিক **নির্দিষ্ট সময়ে** কিছু করতে হবে? AlarmManager use করুন। যেমন: Alarm clock, reminder, scheduled notification।

---

### Key Features:

#### 1. **Exact Time Execution (নির্দিষ্ট সময়)**
```
✅ 6:00 AM ঠিক সময়ে alarm
✅ 2:30 PM এ reminder
✅ Every day 9:00 PM এ notification
```

#### 2. **Survives App Kill & Reboot**
```
✅ App বন্ধ থাকলেও alarm বাজবে
✅ Device restart হলেও alarm থাকবে (with BOOT_COMPLETED)
✅ System manages execution
```

#### 3. **Battery Drain (সতর্কতা!)**
```
⚠️ Device wake up করতে পারে
⚠️ Frequent alarms = battery drain
⚠️ Use carefully!
```

#### 4. **Types of Timing**
```
✅ Exact time (precise)
✅ Inexact time (flexible - battery efficient)
✅ Repeating alarms
```

---

## 2. কেন AlarmManager ব্যবহার করবেন?

### ✅ Use AlarmManager When:

#### 1. **Exact Time Matters (সঠিক সময় গুরুত্বপূর্ণ)**
```
🔔 Alarm clock - 6:00 AM exactly
🔔 Reminder - 2:30 PM exactly
🔔 Medicine reminder - 8:00 AM, 2:00 PM, 8:00 PM
🔔 Prayer time notification
```

#### 2. **User Expects Precision**
```
👤 User set alarm for 6:00 AM
👤 They MUST wake up at 6:00 AM
👤 Not 6:05 AM, not 6:10 AM - EXACTLY 6:00 AM!
```

#### 3. **Scheduled One-time Events**
```
📅 Meeting reminder in 2 hours
📅 Appointment tomorrow at 10:00 AM
📅 Birthday notification on specific date
```

---

### ❌ Don't Use AlarmManager When:

#### 1. **Deferrable Work (পরে করলেও চলবে)**
```java
// ❌ Don't use AlarmManager
// Upload photo - can wait for WiFi
// Sync data - can wait for network
// Clear cache - can wait for charging

// ✅ Use WorkManager instead
```

#### 2. **Background Processing (user doesn't care about timing)**
```java
// ❌ Don't use AlarmManager
// Analytics upload
// Database cleanup
// Log file compression

// ✅ Use WorkManager
```

#### 3. **Foreground Tasks (user is actively using)**
```java
// ❌ Don't use AlarmManager
// Music playback
// Navigation
// Video call

// ✅ Use Foreground Service
```

---

## 3. AlarmManager vs WorkManager

### Quick Comparison:

| Feature | AlarmManager | WorkManager |
|---------|--------------|-------------|
| **Timing** | Exact time ⏰ | Approximate time ⏱️ |
| **Use case** | Alarm, reminder, scheduled events | Background sync, upload, cleanup |
| **Battery** | Can drain 🔋❌ | Optimized 🔋✅ |
| **User expectation** | Must happen at exact time | Can happen later |
| **Example** | 6:00 AM alarm | Upload photos when WiFi available |
| **Doze mode** | Can wake device | Waits for maintenance window |
| **Flexibility** | No constraints | Supports constraints (WiFi, charging) |

---

### Decision Tree:

```
Need to schedule task?
│
├─ Must happen at EXACT time?
│  │
│  ├─ Yes, user expects precision (alarm, reminder)
│  │  └─ Use AlarmManager ✅
│  │
│  └─ No, can be flexible (sync, upload, cleanup)
│     └─ Use WorkManager ✅
│
└─ Need to run periodically?
   │
   ├─ Exact intervals (every 1 hour SHARP)
   │  └─ Use AlarmManager ✅
   │
   └─ Approximate intervals (around every 1 hour)
      └─ Use WorkManager ✅
```

---

## 4. Types of Alarms

### 1️⃣ RTC (Real Time Clock) - Without Wakeup

```kotlin
// Device sleep করলে alarm fire হবে না
// Device wake up হলে তারপর fire হবে

AlarmManager.RTC
```

**Use case:** Non-critical notifications যা device sleep এ দরকার নেই।

**Example:**
```kotlin
// News notification - user যখন phone check করবে তখনই দেখবে
alarmManager.set(
    AlarmManager.RTC,
    triggerTime,
    pendingIntent
)
```

---

### 2️⃣ RTC_WAKEUP - With Wakeup (Most Common)

```kotlin
// Device sleep করলেও wake up করে alarm fire করবে

AlarmManager.RTC_WAKEUP
```

**Use case:** Important alarms যা user কে wake করতে হবে।

**Example:**
```kotlin
// Morning alarm - MUST wake up device
alarmManager.setExact(
    AlarmManager.RTC_WAKEUP,
    triggerTime,
    pendingIntent
)
```

---

### 3️⃣ ELAPSED_REALTIME - Without Wakeup

```kotlin
// Device boot থেকে elapsed time
// Sleep করলে fire হবে না

AlarmManager.ELAPSED_REALTIME
```

**Use case:** Device বন্ধ/চালু এর সাথে sync করতে চান না।

---

### 4️⃣ ELAPSED_REALTIME_WAKEUP - With Wakeup

```kotlin
// Device boot থেকে elapsed time
// Sleep করলেও wake up করবে

AlarmManager.ELAPSED_REALTIME_WAKEUP
```

**Use case:** Boot থেকে specific duration পরে wake up করতে হবে।

**Example:**
```kotlin
// Device start হওয়ার 1 ঘন্টা পরে task run করুন
val triggerTime = SystemClock.elapsedRealtime() + (60 * 60 * 1000)
alarmManager.set(
    AlarmManager.ELAPSED_REALTIME_WAKEUP,
    triggerTime,
    pendingIntent
)
```

---

### Comparison Table:

| Type | Wake Device? | Time Base | Use Case |
|------|--------------|-----------|----------|
| **RTC** | ❌ No | Wall clock (absolute) | Non-urgent notifications |
| **RTC_WAKEUP** | ✅ Yes | Wall clock (absolute) | **Alarms, reminders** |
| **ELAPSED_REALTIME** | ❌ No | Boot time (relative) | Background tasks |
| **ELAPSED_REALTIME_WAKEUP** | ✅ Yes | Boot time (relative) | Boot-relative tasks |

**Most commonly used: RTC_WAKEUP** ⭐

---

## 5. Simple Steps - কিভাবে কাজ করে (Step by Step)

### Overview - মাত্র 4টা Step!

```
Step 1: Permission দিন (Manifest)
Step 2: BroadcastReceiver বানান (Alarm fire হলে কি করবে)
Step 3: Alarm set করুন (AlarmManager দিয়ে)
Step 4: Test করুন!
```

---

### Step 1: Manifest এ Permission যোগ করুন

**কেন লাগবে:** AlarmManager use করতে এবং reboot এর পরে alarm reschedule করতে

```xml
<!-- AndroidManifest.xml -->
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    
    <!-- AlarmManager permission (Android 12+) -->
    <uses-permission android:name="android.permission.SCHEDULE_EXACT_ALARM" />
    
    <!-- Reboot পরে alarm আবার set করার জন্য -->
    <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />
    
    <application>
        <!-- আপনার BroadcastReceiver register করুন -->
        <receiver
            android:name=".AlarmReceiver"
            android:enabled="true"
            android:exported="false" />
    </application>
</manifest>
```

**ব্যাখ্যা:**
- `SCHEDULE_EXACT_ALARM` - নির্দিষ্ট সময়ে alarm set করার জন্য
- `RECEIVE_BOOT_COMPLETED` - Device restart হলে alarm গুলো আবার set করার জন্য
- `<receiver>` - Alarm fire হলে কোন class এ notification যাবে

---

### Step 2: BroadcastReceiver বানান

**কেন লাগবে:** Alarm fire হলে notification দেখানোর জন্য

```kotlin
// AlarmReceiver.kt
class AlarmReceiver : BroadcastReceiver() {
    
    override fun onReceive(context: Context, intent: Intent) {
        // Alarm fire হয়েছে! এখন কি করবেন?
        
        // 1. Intent থেকে data নিন
        val title = intent.getStringExtra("title") ?: "Alarm"
        val message = intent.getStringExtra("message") ?: "Time's up!"
        
        // 2. Notification দেখান
        showNotification(context, title, message)
    }
    
    private fun showNotification(context: Context, title: String, message: String) {
        val notificationManager = context.getSystemService(Context.NOTIFICATION_SERVICE) 
            as NotificationManager
        
        // Android 8.0+ এর জন্য channel create করুন
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            val channel = NotificationChannel(
                "alarm_channel",           // Channel ID
                "Alarms",                  // Channel name
                NotificationManager.IMPORTANCE_HIGH  // High priority
            )
            notificationManager.createNotificationChannel(channel)
        }
        
        // Notification build করুন
        val notification = NotificationCompat.Builder(context, "alarm_channel")
            .setContentTitle(title)
            .setContentText(message)
            .setSmallIcon(android.R.drawable.ic_lock_idle_alarm)  // Alarm icon
            .setPriority(NotificationCompat.PRIORITY_HIGH)
            .setAutoCancel(true)  // Click করলে dismiss হবে
            .build()
        
        // Notification দেখান
        notificationManager.notify(System.currentTimeMillis().toInt(), notification)
    }
}
```

**ব্যাখ্যা:**
1. `onReceive()` - Alarm fire হলে এই method call হয়
2. Intent থেকে title, message নিচ্ছি
3. Notification channel create করছি (Android 8+)
4. Notification show করছি

---

### Step 3: Alarm Set করুন (Activity/Fragment থেকে)

**এটাই main কাজ! এখানে alarm schedule করবেন।**

```kotlin
// MainActivity.kt বা যেকোনো Activity/Fragment
class MainActivity : AppCompatActivity() {
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        // Button click এ alarm set করবো
        findViewById<Button>(R.id.btnSetAlarm).setOnClickListener {
            setAlarmAfter10Seconds()
        }
    }
    
    private fun setAlarmAfter10Seconds() {
        // 1. AlarmManager নিন
        val alarmManager = getSystemService(Context.ALARM_SERVICE) as AlarmManager
        
        // 2. Intent বানান - কোথায় পাঠাবেন?
        val intent = Intent(this, AlarmReceiver::class.java).apply {
            putExtra("title", "Test Alarm")
            putExtra("message", "10 seconds completed!")
        }
        
        // 3. PendingIntent বানান
        val pendingIntent = PendingIntent.getBroadcast(
            this,
            0,  // Request code (unique ID)
            intent,
            PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE
        )
        
        // 4. কখন alarm fire হবে? (10 seconds পরে)
        val triggerTime = System.currentTimeMillis() + (10 * 1000)  // 10 seconds
        
        // 5. Alarm set করুন!
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
            alarmManager.setExactAndAllowWhileIdle(
                AlarmManager.RTC_WAKEUP,  // Type: Wake device
                triggerTime,              // When: 10 seconds later
                pendingIntent             // What: Send to AlarmReceiver
            )
        } else {
            alarmManager.setExact(
                AlarmManager.RTC_WAKEUP,
                triggerTime,
                pendingIntent
            )
        }
        
        Toast.makeText(this, "Alarm set for 10 seconds!", Toast.LENGTH_SHORT).show()
    }
}
```

**ব্যাখ্যা:**
1. **AlarmManager** - System service
2. **Intent** - AlarmReceiver এ data পাঠাচ্ছি
3. **PendingIntent** - Future এ execute হবে
4. **triggerTime** - 10 seconds পরে
5. **setExactAndAllowWhileIdle** - Exact time এ fire হবে

---

### Step 4: Test করুন!

**Run করুন:**
1. App run করুন
2. "Set Alarm" button click করুন
3. 10 seconds wait করুন
4. Notification আসবে! ✅

**যদি notification না আসে:**
- Notification permission দিয়েছেন কিনা check করুন (Android 13+)
- AlarmReceiver manifest এ আছে কিনা check করুন
- Logcat এ error আছে কিনা দেখুন

---

### আরো Examples (Simple):

#### Example A: 5 Minutes পরে Alarm

```kotlin
fun setAlarmAfter5Minutes() {
    val alarmManager = getSystemService(Context.ALARM_SERVICE) as AlarmManager
    
    val intent = Intent(this, AlarmReceiver::class.java).apply {
        putExtra("title", "5 Minutes Up!")
        putExtra("message", "Time to take a break")
    }
    
    val pendingIntent = PendingIntent.getBroadcast(
        this, 1, intent,
        PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE
    )
    
    // 5 minutes = 5 * 60 * 1000 milliseconds
    val triggerTime = System.currentTimeMillis() + (5 * 60 * 1000)
    
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
        alarmManager.setExactAndAllowWhileIdle(
            AlarmManager.RTC_WAKEUP,
            triggerTime,
            pendingIntent
        )
    }
    
    Toast.makeText(this, "Alarm set for 5 minutes", Toast.LENGTH_SHORT).show()
}
```

---

#### Example B: আগামীকাল সকাল 8 টায় Alarm

```kotlin
fun setAlarmForTomorrow8AM() {
    val alarmManager = getSystemService(Context.ALARM_SERVICE) as AlarmManager
    
    val intent = Intent(this, AlarmReceiver::class.java).apply {
        putExtra("title", "Good Morning!")
        putExtra("message", "Time to wake up")
    }
    
    val pendingIntent = PendingIntent.getBroadcast(
        this, 2, intent,
        PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE
    )
    
    // আগামীকাল সকাল 8 টা
    val calendar = Calendar.getInstance().apply {
        add(Calendar.DAY_OF_YEAR, 1)  // আগামীকাল
        set(Calendar.HOUR_OF_DAY, 8)   // 8 AM
        set(Calendar.MINUTE, 0)
        set(Calendar.SECOND, 0)
    }
    
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
        alarmManager.setExactAndAllowWhileIdle(
            AlarmManager.RTC_WAKEUP,
            calendar.timeInMillis,
            pendingIntent
        )
    }
    
    Toast.makeText(this, "Alarm set for tomorrow 8 AM", Toast.LENGTH_SHORT).show()
}
```

---

#### Example C: প্রতিদিন সকাল 7 টায় Repeating Alarm

```kotlin
fun setDailyAlarm7AM() {
    val alarmManager = getSystemService(Context.ALARM_SERVICE) as AlarmManager
    
    val intent = Intent(this, AlarmReceiver::class.java).apply {
        putExtra("title", "Daily Reminder")
        putExtra("message", "Start your day!")
    }
    
    val pendingIntent = PendingIntent.getBroadcast(
        this, 3, intent,
        PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE
    )
    
    // আজকে বা আগামীকাল সকাল 7 টা
    val calendar = Calendar.getInstance().apply {
        set(Calendar.HOUR_OF_DAY, 7)
        set(Calendar.MINUTE, 0)
        set(Calendar.SECOND, 0)
        
        // যদি 7 টা পার হয়ে গেছে, তাহলে আগামীকাল
        if (timeInMillis <= System.currentTimeMillis()) {
            add(Calendar.DAY_OF_YEAR, 1)
        }
    }
    
    // Repeating alarm - প্রতিদিন
    alarmManager.setRepeating(
        AlarmManager.RTC_WAKEUP,
        calendar.timeInMillis,
        AlarmManager.INTERVAL_DAY,  // 24 hours interval
        pendingIntent
    )
    
    Toast.makeText(this, "Daily alarm set for 7 AM", Toast.LENGTH_SHORT).show()
}
```

---

#### Example D: Alarm Cancel করা

```kotlin
fun cancelAlarm(requestCode: Int) {
    val alarmManager = getSystemService(Context.ALARM_SERVICE) as AlarmManager
    
    val intent = Intent(this, AlarmReceiver::class.java)
    val pendingIntent = PendingIntent.getBroadcast(
        this,
        requestCode,  // Same request code যা set করার সময় দিয়েছিলেন
        intent,
        PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE
    )
    
    // Cancel করুন
    alarmManager.cancel(pendingIntent)
    pendingIntent.cancel()
    
    Toast.makeText(this, "Alarm cancelled", Toast.LENGTH_SHORT).show()
}
```

---

### Common Issues এবং Solutions:

#### Issue 1: Notification আসছে না

**Solution:**
```kotlin
// Notification permission check করুন (Android 13+)
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
    if (ContextCompat.checkSelfPermission(
            this,
            Manifest.permission.POST_NOTIFICATIONS
        ) != PackageManager.PERMISSION_GRANTED
    ) {
        ActivityCompat.requestPermissions(
            this,
            arrayOf(Manifest.permission.POST_NOTIFICATIONS),
            100
        )
    }
}
```

---

#### Issue 2: Alarm Doze mode এ fire হচ্ছে না

**Solution:**
```kotlin
// setExact() এর বদলে setExactAndAllowWhileIdle() use করুন
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
    alarmManager.setExactAndAllowWhileIdle(...)
}
```

---

#### Issue 3: Reboot পরে alarm চলে যাচ্ছে

**Solution:**
```kotlin
// BOOT_COMPLETED receiver যোগ করুন
class BootReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        if (intent.action == Intent.ACTION_BOOT_COMPLETED) {
            // Alarms আবার set করুন
            rescheduleAlarms(context)
        }
    }
}

// Manifest এ
<receiver android:name=".BootReceiver">
    <intent-filter>
        <action android:name="android.intent.action.BOOT_COMPLETED" />
    </intent-filter>
</receiver>
```

---

## 6. Complete Working Example (Minimal)

### একদম সহজ, minimal example - Copy করলেই কাজ করবে!

#### File 1: AndroidManifest.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    
    <uses-permission android:name="android.permission.SCHEDULE_EXACT_ALARM" />
    <uses-permission android:name="android.permission.POST_NOTIFICATIONS" />
    
    <application
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name">
        
        <activity
            android:name=".MainActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        
        <receiver
            android:name=".AlarmReceiver"
            android:enabled="true"
            android:exported="false" />
    </application>
</manifest>
```

---

#### File 2: AlarmReceiver.kt

```kotlin
package com.example.alarmexample

import android.app.NotificationChannel
import android.app.NotificationManager
import android.content.BroadcastReceiver
import android.content.Context
import android.content.Intent
import android.os.Build
import androidx.core.app.NotificationCompat

class AlarmReceiver : BroadcastReceiver() {
    
    override fun onReceive(context: Context, intent: Intent) {
        val title = intent.getStringExtra("title") ?: "Alarm"
        val message = intent.getStringExtra("message") ?: "Time's up!"
        
        showNotification(context, title, message)
    }
    
    private fun showNotification(context: Context, title: String, message: String) {
        val notificationManager = context.getSystemService(Context.NOTIFICATION_SERVICE) 
            as NotificationManager
        
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            val channel = NotificationChannel(
                "alarm_channel",
                "Alarms",
                NotificationManager.IMPORTANCE_HIGH
            )
            notificationManager.createNotificationChannel(channel)
        }
        
        val notification = NotificationCompat.Builder(context, "alarm_channel")
            .setContentTitle(title)
            .setContentText(message)
            .setSmallIcon(android.R.drawable.ic_lock_idle_alarm)
            .setPriority(NotificationCompat.PRIORITY_HIGH)
            .setAutoCancel(true)
            .build()
        
        notificationManager.notify(System.currentTimeMillis().toInt(), notification)
    }
}
```

---

#### File 3: MainActivity.kt

```kotlin
package com.example.alarmexample

import android.app.AlarmManager
import android.app.PendingIntent
import android.content.Context
import android.content.Intent
import android.os.Build
import android.os.Bundle
import android.widget.Button
import android.widget.Toast
import androidx.appcompat.app.AppCompatActivity

class MainActivity : AppCompatActivity() {
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        findViewById<Button>(R.id.btnSetAlarm).setOnClickListener {
            setAlarmAfter10Seconds()
        }
    }
    
    private fun setAlarmAfter10Seconds() {
        val alarmManager = getSystemService(Context.ALARM_SERVICE) as AlarmManager
        
        val intent = Intent(this, AlarmReceiver::class.java).apply {
            putExtra("title", "Test Alarm")
            putExtra("message", "10 seconds completed!")
        }
        
        val pendingIntent = PendingIntent.getBroadcast(
            this, 0, intent,
            PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE
        )
        
        val triggerTime = System.currentTimeMillis() + (10 * 1000)
        
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
            alarmManager.setExactAndAllowWhileIdle(
                AlarmManager.RTC_WAKEUP,
                triggerTime,
                pendingIntent
            )
        } else {
            alarmManager.setExact(
                AlarmManager.RTC_WAKEUP,
                triggerTime,
                pendingIntent
            )
        }
        
        Toast.makeText(this, "Alarm set for 10 seconds!", Toast.LENGTH_SHORT).show()
    }
}
```

---

#### File 4: activity_main.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:gravity="center"
    android:orientation="vertical"
    android:padding="16dp">
    
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="AlarmManager Example"
        android:textSize="20sp"
        android:textStyle="bold" />
    
    <Button
        android:id="@+id/btnSetAlarm"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="16dp"
        android:text="Set Alarm (10 seconds)" />
</LinearLayout>
```

---

### এই example এ কি আছে:

✅ **4টা file** - Manifest, Receiver, Activity, Layout  
✅ **মাত্র 10 seconds** এ alarm fire হবে  
✅ **Notification** দেখাবে  
✅ **Copy-paste ready** - সরাসরি কাজ করবে  

---

## 7. Real-world Example 1: Reminder App

এখন আসল application এ দেখি কিভাবে use করবেন।

**Scenario:**
User একটা reminder set করতে চায় - "Call Mom at 2:30 PM tomorrow"

এখন পুরো reminder app বানাবো - database, UI, notification সব সহ।

---

### Complete Implementation:

#### 1. Data Model

```kotlin
data class Reminder(
    val id: Int,
    val title: String,
    val message: String,
    val timeInMillis: Long,
    val isActive: Boolean = true
)
```

---

#### 2. ReminderManager Class

```kotlin
class ReminderManager(private val context: Context) {
    
    private val alarmManager = context.getSystemService(Context.ALARM_SERVICE) as AlarmManager
    private val prefs = context.getSharedPreferences("reminders", Context.MODE_PRIVATE)
    
    fun setReminder(reminder: Reminder) {
        // Save reminder to preferences
        saveReminder(reminder)
        
        // Schedule alarm
        scheduleAlarm(reminder)
    }
    
    private fun scheduleAlarm(reminder: Reminder) {
        val intent = Intent(context, ReminderReceiver::class.java).apply {
            action = "ACTION_REMINDER"
            putExtra("reminder_id", reminder.id)
            putExtra("title", reminder.title)
            putExtra("message", reminder.message)
        }
        
        val pendingIntent = PendingIntent.getBroadcast(
            context,
            reminder.id,
            intent,
            PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE
        )
        
        // Schedule exact alarm
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
            // Android 6.0+ - Use setExactAndAllowWhileIdle for Doze mode
            alarmManager.setExactAndAllowWhileIdle(
                AlarmManager.RTC_WAKEUP,
                reminder.timeInMillis,
                pendingIntent
            )
        } else {
            alarmManager.setExact(
                AlarmManager.RTC_WAKEUP,
                reminder.timeInMillis,
                pendingIntent
            )
        }
        
        Log.d("ReminderManager", "Reminder scheduled for ${Date(reminder.timeInMillis)}")
    }
    
    fun cancelReminder(reminderId: Int) {
        // Remove from preferences
        removeReminder(reminderId)
        
        // Cancel alarm
        val intent = Intent(context, ReminderReceiver::class.java)
        val pendingIntent = PendingIntent.getBroadcast(
            context,
            reminderId,
            intent,
            PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE
        )
        
        alarmManager.cancel(pendingIntent)
        pendingIntent.cancel()
        
        Log.d("ReminderManager", "Reminder $reminderId cancelled")
    }
    
    fun rescheduleAllReminders() {
        val reminders = getAllReminders()
        reminders.forEach { reminder ->
            if (reminder.isActive && reminder.timeInMillis > System.currentTimeMillis()) {
                scheduleAlarm(reminder)
            }
        }
    }
    
    private fun saveReminder(reminder: Reminder) {
        val json = Gson().toJson(reminder)
        prefs.edit().putString("reminder_${reminder.id}", json).apply()
    }
    
    private fun removeReminder(reminderId: Int) {
        prefs.edit().remove("reminder_$reminderId").apply()
    }
    
    private fun getAllReminders(): List<Reminder> {
        val reminders = mutableListOf<Reminder>()
        prefs.all.forEach { (key, value) ->
            if (key.startsWith("reminder_")) {
                val reminder = Gson().fromJson(value as String, Reminder::class.java)
                reminders.add(reminder)
            }
        }
        return reminders
    }
}
```

---

#### 3. ReminderReceiver

```kotlin
class ReminderReceiver : BroadcastReceiver() {
    
    override fun onReceive(context: Context, intent: Intent) {
        when (intent.action) {
            "ACTION_REMINDER" -> {
                val reminderId = intent.getIntExtra("reminder_id", -1)
                val title = intent.getStringExtra("title") ?: "Reminder"
                val message = intent.getStringExtra("message") ?: ""
                
                showReminderNotification(context, reminderId, title, message)
                
                // Play notification sound
                playNotificationSound(context)
            }
            
            Intent.ACTION_BOOT_COMPLETED -> {
                // Reschedule all reminders after reboot
                val reminderManager = ReminderManager(context)
                reminderManager.rescheduleAllReminders()
            }
        }
    }
    
    private fun showReminderNotification(
        context: Context,
        reminderId: Int,
        title: String,
        message: String
    ) {
        val notificationManager = context.getSystemService(Context.NOTIFICATION_SERVICE) 
            as NotificationManager
        
        // Create notification channel (Android 8+)
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            val channel = NotificationChannel(
                "reminder_channel",
                "Reminders",
                NotificationManager.IMPORTANCE_HIGH
            ).apply {
                description = "Reminder notifications"
                enableVibration(true)
                enableLights(true)
            }
            notificationManager.createNotificationChannel(channel)
        }
        
        // Create intent to open app when notification clicked
        val openIntent = Intent(context, MainActivity::class.java).apply {
            flags = Intent.FLAG_ACTIVITY_NEW_TASK or Intent.FLAG_ACTIVITY_CLEAR_TASK
            putExtra("reminder_id", reminderId)
        }
        
        val pendingIntent = PendingIntent.getActivity(
            context,
            reminderId,
            openIntent,
            PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE
        )
        
        // Build notification
        val notification = NotificationCompat.Builder(context, "reminder_channel")
            .setContentTitle(title)
            .setContentText(message)
            .setSmallIcon(R.drawable.ic_reminder)
            .setPriority(NotificationCompat.PRIORITY_HIGH)
            .setCategory(NotificationCompat.CATEGORY_REMINDER)
            .setAutoCancel(true)
            .setContentIntent(pendingIntent)
            .setVibrate(longArrayOf(0, 500, 200, 500))
            .build()
        
        notificationManager.notify(reminderId, notification)
    }
    
    private fun playNotificationSound(context: Context) {
        val notification = RingtoneManager.getDefaultUri(RingtoneManager.TYPE_NOTIFICATION)
        val ringtone = RingtoneManager.getRingtone(context, notification)
        ringtone.play()
    }
}
```

---

#### 4. UI - Setting Reminder (Activity/Fragment)

```kotlin
class SetReminderActivity : AppCompatActivity() {
    
    private lateinit var reminderManager: ReminderManager
    private var selectedTimeInMillis: Long = 0
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_set_reminder)
        
        reminderManager = ReminderManager(this)
        
        // Date/Time picker
        val btnPickDateTime = findViewById<Button>(R.id.btnPickDateTime)
        val etTitle = findViewById<EditText>(R.id.etTitle)
        val etMessage = findViewById<EditText>(R.id.etMessage)
        val btnSetReminder = findViewById<Button>(R.id.btnSetReminder)
        
        btnPickDateTime.setOnClickListener {
            showDateTimePicker()
        }
        
        btnSetReminder.setOnClickListener {
            val title = etTitle.text.toString()
            val message = etMessage.text.toString()
            
            if (title.isNotEmpty() && selectedTimeInMillis > System.currentTimeMillis()) {
                setReminder(title, message, selectedTimeInMillis)
            } else {
                Toast.makeText(this, "Please fill all fields", Toast.LENGTH_SHORT).show()
            }
        }
    }
    
    private fun showDateTimePicker() {
        val calendar = Calendar.getInstance()
        
        // Date picker
        DatePickerDialog(
            this,
            { _, year, month, dayOfMonth ->
                calendar.set(Calendar.YEAR, year)
                calendar.set(Calendar.MONTH, month)
                calendar.set(Calendar.DAY_OF_MONTH, dayOfMonth)
                
                // Time picker
                TimePickerDialog(
                    this,
                    { _, hourOfDay, minute ->
                        calendar.set(Calendar.HOUR_OF_DAY, hourOfDay)
                        calendar.set(Calendar.MINUTE, minute)
                        calendar.set(Calendar.SECOND, 0)
                        
                        selectedTimeInMillis = calendar.timeInMillis
                        
                        val dateFormat = SimpleDateFormat("MMM dd, yyyy hh:mm a", Locale.getDefault())
                        findViewById<Button>(R.id.btnPickDateTime).text = 
                            dateFormat.format(Date(selectedTimeInMillis))
                    },
                    calendar.get(Calendar.HOUR_OF_DAY),
                    calendar.get(Calendar.MINUTE),
                    false
                ).show()
            },
            calendar.get(Calendar.YEAR),
            calendar.get(Calendar.MONTH),
            calendar.get(Calendar.DAY_OF_MONTH)
        ).apply {
            datePicker.minDate = System.currentTimeMillis()
        }.show()
    }
    
    private fun setReminder(title: String, message: String, timeInMillis: Long) {
        val reminder = Reminder(
            id = System.currentTimeMillis().toInt(),
            title = title,
            message = message,
            timeInMillis = timeInMillis,
            isActive = true
        )
        
        reminderManager.setReminder(reminder)
        
        val dateFormat = SimpleDateFormat("MMM dd, yyyy hh:mm a", Locale.getDefault())
        Toast.makeText(
            this,
            "Reminder set for ${dateFormat.format(Date(timeInMillis))}",
            Toast.LENGTH_LONG
        ).show()
        
        finish()
    }
}
```

---

## 8. Real-world Example 2: Medicine Reminder

### Scenario:
User কে প্রতিদিন ৩ বার (সকাল ৮টা, দুপুর ২টা, রাত ৮টা) medicine খাওয়ার reminder দিতে হবে।

---

### Complete Implementation:

#### 1. Data Model

```kotlin
data class MedicineReminder(
    val id: Int,
    val medicineName: String,
    val dosage: String,
    val times: List<String>, // ["08:00", "14:00", "20:00"]
    val isActive: Boolean = true
)

data class MedicineSchedule(
    val id: Int,
    val medicineId: Int,
    val medicineName: String,
    val dosage: String,
    val timeInMillis: Long,
    val timeString: String // "08:00"
)
```

---

#### 2. MedicineReminderManager

```kotlin
class MedicineReminderManager(private val context: Context) {
    
    private val alarmManager = context.getSystemService(Context.ALARM_SERVICE) as AlarmManager
    
    fun scheduleMedicineReminder(reminder: MedicineReminder) {
        // Schedule alarm for each time
        reminder.times.forEachIndexed { index, timeString ->
            val scheduleId = generateScheduleId(reminder.id, index)
            val timeInMillis = calculateNextAlarmTime(timeString)
            
            val schedule = MedicineSchedule(
                id = scheduleId,
                medicineId = reminder.id,
                medicineName = reminder.medicineName,
                dosage = reminder.dosage,
                timeInMillis = timeInMillis,
                timeString = timeString
            )
            
            scheduleRepeatingAlarm(schedule)
        }
    }
    
    private fun scheduleRepeatingAlarm(schedule: MedicineSchedule) {
        val intent = Intent(context, MedicineReminderReceiver::class.java).apply {
            action = "ACTION_MEDICINE_REMINDER"
            putExtra("schedule_id", schedule.id)
            putExtra("medicine_id", schedule.medicineId)
            putExtra("medicine_name", schedule.medicineName)
            putExtra("dosage", schedule.dosage)
            putExtra("time", schedule.timeString)
        }
        
        val pendingIntent = PendingIntent.getBroadcast(
            context,
            schedule.id,
            intent,
            PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE
        )
        
        // Set repeating alarm - every 24 hours
        alarmManager.setRepeating(
            AlarmManager.RTC_WAKEUP,
            schedule.timeInMillis,
            AlarmManager.INTERVAL_DAY, // 24 hours
            pendingIntent
        )
        
        Log.d("MedicineReminder", 
            "Scheduled ${schedule.medicineName} at ${schedule.timeString}")
    }
    
    fun cancelMedicineReminder(medicineId: Int, timesCount: Int) {
        // Cancel all scheduled times for this medicine
        for (index in 0 until timesCount) {
            val scheduleId = generateScheduleId(medicineId, index)
            
            val intent = Intent(context, MedicineReminderReceiver::class.java)
            val pendingIntent = PendingIntent.getBroadcast(
                context,
                scheduleId,
                intent,
                PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE
            )
            
            alarmManager.cancel(pendingIntent)
            pendingIntent.cancel()
        }
        
        Log.d("MedicineReminder", "Cancelled medicine reminder: $medicineId")
    }
    
    private fun calculateNextAlarmTime(timeString: String): Long {
        // timeString format: "HH:mm" (e.g., "08:00")
        val parts = timeString.split(":")
        val hour = parts[0].toInt()
        val minute = parts[1].toInt()
        
        val calendar = Calendar.getInstance().apply {
            set(Calendar.HOUR_OF_DAY, hour)
            set(Calendar.MINUTE, minute)
            set(Calendar.SECOND, 0)
            set(Calendar.MILLISECOND, 0)
        }
        
        // If time has passed today, schedule for tomorrow
        if (calendar.timeInMillis <= System.currentTimeMillis()) {
            calendar.add(Calendar.DAY_OF_YEAR, 1)
        }
        
        return calendar.timeInMillis
    }
    
    private fun generateScheduleId(medicineId: Int, index: Int): Int {
        // Generate unique ID: medicineId * 100 + index
        return medicineId * 100 + index
    }
}
```

---

#### 3. MedicineReminderReceiver

```kotlin
class MedicineReminderReceiver : BroadcastReceiver() {
    
    override fun onReceive(context: Context, intent: Intent) {
        when (intent.action) {
            "ACTION_MEDICINE_REMINDER" -> {
                val scheduleId = intent.getIntExtra("schedule_id", -1)
                val medicineName = intent.getStringExtra("medicine_name") ?: "Medicine"
                val dosage = intent.getStringExtra("dosage") ?: ""
                val time = intent.getStringExtra("time") ?: ""
                
                showMedicineNotification(context, scheduleId, medicineName, dosage, time)
            }
            
            Intent.ACTION_BOOT_COMPLETED -> {
                // Reschedule all medicine reminders after reboot
                rescheduleMedicineReminders(context)
            }
        }
    }
    
    private fun showMedicineNotification(
        context: Context,
        scheduleId: Int,
        medicineName: String,
        dosage: String,
        time: String
    ) {
        val notificationManager = context.getSystemService(Context.NOTIFICATION_SERVICE) 
            as NotificationManager
        
        // Create notification channel
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            val channel = NotificationChannel(
                "medicine_channel",
                "Medicine Reminders",
                NotificationManager.IMPORTANCE_HIGH
            ).apply {
                description = "Daily medicine reminders"
                enableVibration(true)
                setSound(
                    RingtoneManager.getDefaultUri(RingtoneManager.TYPE_NOTIFICATION),
                    null
                )
            }
            notificationManager.createNotificationChannel(channel)
        }
        
        // Create "Taken" action
        val takenIntent = Intent(context, MedicineActionReceiver::class.java).apply {
            action = "ACTION_MEDICINE_TAKEN"
            putExtra("schedule_id", scheduleId)
            putExtra("medicine_name", medicineName)
        }
        val takenPendingIntent = PendingIntent.getBroadcast(
            context,
            scheduleId * 10 + 1,
            takenIntent,
            PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE
        )
        
        // Create "Snooze" action
        val snoozeIntent = Intent(context, MedicineActionReceiver::class.java).apply {
            action = "ACTION_MEDICINE_SNOOZE"
            putExtra("schedule_id", scheduleId)
            putExtra("medicine_name", medicineName)
            putExtra("dosage", dosage)
            putExtra("time", time)
        }
        val snoozePendingIntent = PendingIntent.getBroadcast(
            context,
            scheduleId * 10 + 2,
            snoozeIntent,
            PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE
        )
        
        // Build notification
        val notification = NotificationCompat.Builder(context, "medicine_channel")
            .setContentTitle("💊 $medicineName")
            .setContentText("Time to take your medicine ($dosage) at $time")
            .setSmallIcon(R.drawable.ic_medicine)
            .setPriority(NotificationCompat.PRIORITY_HIGH)
            .setCategory(NotificationCompat.CATEGORY_REMINDER)
            .setAutoCancel(true)
            .addAction(
                R.drawable.ic_check,
                "Taken",
                takenPendingIntent
            )
            .addAction(
                R.drawable.ic_snooze,
                "Snooze 15 min",
                snoozePendingIntent
            )
            .setVibrate(longArrayOf(0, 500, 200, 500, 200, 500))
            .build()
        
        notificationManager.notify(scheduleId, notification)
        
        // Play sound
        val ringtone = RingtoneManager.getRingtone(
            context,
            RingtoneManager.getDefaultUri(RingtoneManager.TYPE_NOTIFICATION)
        )
        ringtone.play()
    }
    
    private fun rescheduleMedicineReminders(context: Context) {
        // Load saved medicine reminders from database/preferences
        // and reschedule them
        val medicineDb = MedicineDatabase.getInstance(context)
        val reminders = medicineDb.medicineDao().getAllActiveReminders()
        
        val manager = MedicineReminderManager(context)
        reminders.forEach { reminder ->
            manager.scheduleMedicineReminder(reminder)
        }
    }
}
```

---

#### 4. MedicineActionReceiver (Handle "Taken" and "Snooze")

```kotlin
class MedicineActionReceiver : BroadcastReceiver() {
    
    override fun onReceive(context: Context, intent: Intent) {
        val scheduleId = intent.getIntExtra("schedule_id", -1)
        val medicineName = intent.getStringExtra("medicine_name") ?: ""
        
        when (intent.action) {
            "ACTION_MEDICINE_TAKEN" -> {
                handleMedicineTaken(context, scheduleId, medicineName)
            }
            
            "ACTION_MEDICINE_SNOOZE" -> {
                val dosage = intent.getStringExtra("dosage") ?: ""
                val time = intent.getStringExtra("time") ?: ""
                handleMedicineSnooze(context, scheduleId, medicineName, dosage, time)
            }
        }
    }
    
    private fun handleMedicineTaken(context: Context, scheduleId: Int, medicineName: String) {
        // Dismiss notification
        val notificationManager = context.getSystemService(Context.NOTIFICATION_SERVICE) 
            as NotificationManager
        notificationManager.cancel(scheduleId)
        
        // Log to database
        val timestamp = System.currentTimeMillis()
        logMedicineTaken(context, scheduleId, medicineName, timestamp)
        
        // Show confirmation toast
        Toast.makeText(
            context,
            "✓ $medicineName marked as taken",
            Toast.LENGTH_SHORT
        ).show()
    }
    
    private fun handleMedicineSnooze(
        context: Context,
        scheduleId: Int,
        medicineName: String,
        dosage: String,
        time: String
    ) {
        // Dismiss current notification
        val notificationManager = context.getSystemService(Context.NOTIFICATION_SERVICE) 
            as NotificationManager
        notificationManager.cancel(scheduleId)
        
        // Schedule snooze alarm (15 minutes later)
        val alarmManager = context.getSystemService(Context.ALARM_SERVICE) as AlarmManager
        val snoozeTime = System.currentTimeMillis() + (15 * 60 * 1000) // 15 minutes
        
        val snoozeIntent = Intent(context, MedicineReminderReceiver::class.java).apply {
            action = "ACTION_MEDICINE_REMINDER"
            putExtra("schedule_id", scheduleId)
            putExtra("medicine_name", medicineName)
            putExtra("dosage", dosage)
            putExtra("time", time)
        }
        
        val pendingIntent = PendingIntent.getBroadcast(
            context,
            scheduleId + 1000, // Different request code for snooze
            snoozeIntent,
            PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE
        )
        
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
            alarmManager.setExactAndAllowWhileIdle(
                AlarmManager.RTC_WAKEUP,
                snoozeTime,
                pendingIntent
            )
        } else {
            alarmManager.setExact(
                AlarmManager.RTC_WAKEUP,
                snoozeTime,
                pendingIntent
            )
        }
        
        Toast.makeText(
            context,
            "Snoozed for 15 minutes",
            Toast.LENGTH_SHORT
        ).show()
    }
    
    private fun logMedicineTaken(
        context: Context,
        scheduleId: Int,
        medicineName: String,
        timestamp: Long
    ) {
        // Save to database
        val db = MedicineDatabase.getInstance(context)
        val log = MedicineLog(
            scheduleId = scheduleId,
            medicineName = medicineName,
            takenAt = timestamp
        )
        db.medicineDao().insertLog(log)
    }
}
```

---

#### 5. UI - Add Medicine Reminder

```kotlin
class AddMedicineActivity : AppCompatActivity() {
    
    private lateinit var medicineManager: MedicineReminderManager
    private val selectedTimes = mutableListOf<String>()
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_add_medicine)
        
        medicineManager = MedicineReminderManager(this)
        
        val etMedicineName = findViewById<EditText>(R.id.etMedicineName)
        val etDosage = findViewById<EditText>(R.id.etDosage)
        val btnAddTime = findViewById<Button>(R.id.btnAddTime)
        val rvTimes = findViewById<RecyclerView>(R.id.rvTimes)
        val btnSave = findViewById<Button>(R.id.btnSave)
        
        // Setup RecyclerView for selected times
        val adapter = TimesAdapter(selectedTimes) { position ->
            selectedTimes.removeAt(position)
            rvTimes.adapter?.notifyItemRemoved(position)
        }
        rvTimes.adapter = adapter
        rvTimes.layoutManager = LinearLayoutManager(this)
        
        // Add time button
        btnAddTime.setOnClickListener {
            showTimePicker { time ->
                selectedTimes.add(time)
                adapter.notifyItemInserted(selectedTimes.size - 1)
            }
        }
        
        // Save button
        btnSave.setOnClickListener {
            val medicineName = etMedicineName.text.toString()
            val dosage = etDosage.text.toString()
            
            if (medicineName.isNotEmpty() && dosage.isNotEmpty() && selectedTimes.isNotEmpty()) {
                saveMedicineReminder(medicineName, dosage, selectedTimes)
            } else {
                Toast.makeText(this, "Please fill all fields", Toast.LENGTH_SHORT).show()
            }
        }
    }
    
    private fun showTimePicker(onTimeSelected: (String) -> Unit) {
        val calendar = Calendar.getInstance()
        
        TimePickerDialog(
            this,
            { _, hourOfDay, minute ->
                val timeString = String.format("%02d:%02d", hourOfDay, minute)
                onTimeSelected(timeString)
            },
            calendar.get(Calendar.HOUR_OF_DAY),
            calendar.get(Calendar.MINUTE),
            true // 24-hour format
        ).show()
    }
    
    private fun saveMedicineReminder(
        medicineName: String,
        dosage: String,
        times: List<String>
    ) {
        val reminder = MedicineReminder(
            id = System.currentTimeMillis().toInt(),
            medicineName = medicineName,
            dosage = dosage,
            times = times,
            isActive = true
        )
        
        // Save to database
        val db = MedicineDatabase.getInstance(this)
        db.medicineDao().insert(reminder)
        
        // Schedule alarms
        medicineManager.scheduleMedicineReminder(reminder)
        
        Toast.makeText(
            this,
            "Medicine reminder set for ${times.size} times daily",
            Toast.LENGTH_LONG
        ).show()
        
        finish()
    }
}
```

---

## 9. Best Practices

### ✅ DO

#### 1. **Use setExactAndAllowWhileIdle for Doze Mode**

```kotlin
// ✅ Good - Works in Doze mode
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
    alarmManager.setExactAndAllowWhileIdle(
        AlarmManager.RTC_WAKEUP,
        triggerTime,
        pendingIntent
    )
}

// ❌ Bad - May not work in Doze mode
alarmManager.setExact(
    AlarmManager.RTC_WAKEUP,
    triggerTime,
    pendingIntent
)
```

---

#### 2. **Check Exact Alarm Permission (Android 12+)**

```kotlin
// ✅ Good - Check permission
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.S) {
    if (alarmManager.canScheduleExactAlarms()) {
        // Schedule alarm
    } else {
        // Request permission
        val intent = Intent(Settings.ACTION_REQUEST_SCHEDULE_EXACT_ALARM)
        startActivity(intent)
    }
}
```

---

#### 3. **Handle BOOT_COMPLETED to Reschedule**

```kotlin
// ✅ Good - Reschedule after reboot
class AlarmReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        if (intent.action == Intent.ACTION_BOOT_COMPLETED) {
            rescheduleAllAlarms(context)
        }
    }
}

// AndroidManifest.xml
<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />
<receiver android:name=".AlarmReceiver">
    <intent-filter>
        <action android:name="android.intent.action.BOOT_COMPLETED" />
    </intent-filter>
</receiver>
```

---

#### 4. **Use Unique Request Codes**

```kotlin
// ✅ Good - Unique request codes
alarms.forEach { alarm ->
    val pendingIntent = PendingIntent.getBroadcast(
        context,
        alarm.id, // Unique ID
        intent,
        PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE
    )
}

// ❌ Bad - Same request code
val pendingIntent = PendingIntent.getBroadcast(
    context,
    0, // Same for all alarms!
    intent,
    PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE
)
```

---

#### 5. **Save Alarm Data Persistently**

```kotlin
// ✅ Good - Save to database/preferences
fun setAlarm(alarm: Alarm) {
    // Save to database
    database.alarmDao().insert(alarm)
    
    // Schedule alarm
    scheduleAlarm(alarm)
}

// ❌ Bad - Only in memory
fun setAlarm(alarm: Alarm) {
    // Only schedule, don't save
    scheduleAlarm(alarm)
    // Lost after reboot!
}
```

---

### ❌ DON'T

#### 1. **Don't Use for Deferrable Work**

```kotlin
// ❌ Bad - Using AlarmManager for upload
alarmManager.setRepeating(
    AlarmManager.RTC_WAKEUP,
    System.currentTimeMillis(),
    15 * 60 * 1000, // Every 15 minutes
    uploadPendingIntent
)
// Battery drain! Use WorkManager instead

// ✅ Good - Use WorkManager
val work = PeriodicWorkRequestBuilder<UploadWorker>(
    15, TimeUnit.MINUTES
).build()
WorkManager.getInstance(context).enqueue(work)
```

---

#### 2. **Don't Schedule Too Frequent Alarms**

```kotlin
// ❌ Bad - Every minute!
alarmManager.setRepeating(
    AlarmManager.RTC_WAKEUP,
    triggerTime,
    60 * 1000, // Every 1 minute - BAD!
    pendingIntent
)
// Battery will drain fast!

// ✅ Good - Use Foreground Service or Handler
class MyService : Service() {
    private val handler = Handler(Looper.getMainLooper())
    private val runnable = object : Runnable {
        override fun run() {
            doWork()
            handler.postDelayed(this, 60 * 1000)
        }
    }
}
```

---

#### 3. **Don't Forget to Cancel**

```kotlin
// ✅ Good - Cancel when not needed
fun cancelAlarm(alarmId: Int) {
    val intent = Intent(context, AlarmReceiver::class.java)
    val pendingIntent = PendingIntent.getBroadcast(
        context,
        alarmId,
        intent,
        PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE
    )
    
    alarmManager.cancel(pendingIntent)
    pendingIntent.cancel() // Important!
}

// ❌ Bad - Only cancel alarm, not PendingIntent
alarmManager.cancel(pendingIntent)
// PendingIntent still exists!
```

---

## 10. Interview Questions

### Q1: AlarmManager কি এবং কখন use করবেন?

**Answer:**

**Definition:**  
AlarmManager হলো Android system service যা **specific time** বা **interval** এ task execute করার জন্য।

**Key Features:**
- Exact time execution
- Survives app kill & reboot
- Can wake device from sleep
- System-managed

**কখন use করবেন:**

**✅ Use AlarmManager:**
```
- Alarm clock (6:00 AM exact)
- Reminders (user expects exact time)
- Medicine reminders (8:00 AM, 2:00 PM, 8:00 PM)
- Prayer time notifications
- Scheduled one-time events
```

**❌ Don't use AlarmManager:**
```
- Background data sync (use WorkManager)
- File upload/download (use WorkManager)
- Cache cleanup (use WorkManager)
- Analytics upload (use WorkManager)
```

**Example:**
```kotlin
// User set alarm for 6:00 AM
val calendar = Calendar.getInstance().apply {
    set(Calendar.HOUR_OF_DAY, 6)
    set(Calendar.MINUTE, 0)
    set(Calendar.SECOND, 0)
}

alarmManager.setExact(
    AlarmManager.RTC_WAKEUP,
    calendar.timeInMillis,
    pendingIntent
)
```

**Key Point:**  
AlarmManager = **Exact time matters**, user expects precision.  
WorkManager = **Approximate time OK**, system decides best time.

---

### Q2: AlarmManager এর different types কি কি?

**Answer:**

**4 Types of Alarms:**

**1. RTC (Real Time Clock):**
```kotlin
AlarmManager.RTC
// - Wall clock time (absolute)
// - Does NOT wake device
// - Use: Non-urgent notifications
```

**2. RTC_WAKEUP (Most Common):**
```kotlin
AlarmManager.RTC_WAKEUP
// - Wall clock time (absolute)
// - WAKES device from sleep
// - Use: Alarms, important reminders
```

**3. ELAPSED_REALTIME:**
```kotlin
AlarmManager.ELAPSED_REALTIME
// - Boot time (relative)
// - Does NOT wake device
// - Use: Background tasks
```

**4. ELAPSED_REALTIME_WAKEUP:**
```kotlin
AlarmManager.ELAPSED_REALTIME_WAKEUP
// - Boot time (relative)
// - WAKES device from sleep
// - Use: Boot-relative urgent tasks
```

**Comparison:**

| Type | Wake Device? | Time Base | Example |
|------|--------------|-----------|---------|
| RTC | ❌ No | Absolute | News notification |
| **RTC_WAKEUP** | ✅ Yes | Absolute | **Alarm clock** |
| ELAPSED_REALTIME | ❌ No | Relative | Background sync |
| ELAPSED_REALTIME_WAKEUP | ✅ Yes | Relative | Urgent boot task |

**Real Example:**

```kotlin
// Morning alarm - MUST wake device
alarmManager.setExact(
    AlarmManager.RTC_WAKEUP, // Wake device
    triggerTime,
    pendingIntent
)

// News notification - can wait
alarmManager.set(
    AlarmManager.RTC, // Don't wake
    triggerTime,
    pendingIntent
)
```

---

### Q3: setExact() এবং setExactAndAllowWhileIdle() এর পার্থক্য কি?

**Answer:**

**Doze Mode Impact:**

**setExact():**
```kotlin
alarmManager.setExact(
    AlarmManager.RTC_WAKEUP,
    triggerTime,
    pendingIntent
)
// ⚠️ Problem: May NOT fire in Doze mode!
// - Device in deep sleep
// - Alarm delayed until maintenance window
// - Not guaranteed to fire at exact time
```

**setExactAndAllowWhileIdle() (API 23+):**
```kotlin
alarmManager.setExactAndAllowWhileIdle(
    AlarmManager.RTC_WAKEUP,
    triggerTime,
    pendingIntent
)
// ✅ Better: Fires even in Doze mode!
// - Wakes device from deep sleep
// - Guaranteed to fire at exact time
// - Limited to ~9 alarms per hour per app
```

**When to Use:**

| Method | Doze Mode | Use Case |
|--------|-----------|----------|
| **setExact()** | May delay | Less critical alarms |
| **setExactAndAllowWhileIdle()** | Always fires | **Critical alarms** (morning alarm) |

**Best Practice:**

```kotlin
// ✅ Good - Handle both cases
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
    alarmManager.setExactAndAllowWhileIdle(
        AlarmManager.RTC_WAKEUP,
        triggerTime,
        pendingIntent
    )
} else {
    alarmManager.setExact(
        AlarmManager.RTC_WAKEUP,
        triggerTime,
        pendingIntent
    )
}
```

**Limitation:**
- `setExactAndAllowWhileIdle()` - Maximum ~9 alarms per hour per app
- Exceeding limit = System may delay alarms

---

### Q4: Android 12+ এ exact alarm schedule করতে কি permission লাগে?

**Answer:**

**Android 12 (API 31+) Changes:**

**New Permissions Required:**

**1. SCHEDULE_EXACT_ALARM (Automatic):**
```xml
<!-- AndroidManifest.xml -->
<uses-permission android:name="android.permission.SCHEDULE_EXACT_ALARM" />
```
- Automatically granted
- No user approval needed
- For non-clock apps

**2. USE_EXACT_ALARM (Automatic - Clock Apps):**
```xml
<uses-permission android:name="android.permission.USE_EXACT_ALARM" />
```
- For clock/alarm apps only
- Automatically granted
- No revocation by user

---

**Runtime Check Required:**

```kotlin
fun scheduleAlarm() {
    val alarmManager = getSystemService(Context.ALARM_SERVICE) as AlarmManager
    
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.S) {
        // Android 12+: Check permission
        if (alarmManager.canScheduleExactAlarms()) {
            // Permission granted - schedule alarm
            alarmManager.setExact(...)
        } else {
            // Permission not granted - request
            requestExactAlarmPermission()
        }
    } else {
        // Android 11 and below - no permission needed
        alarmManager.setExact(...)
    }
}

@RequiresApi(Build.VERSION_CODES.S)
fun requestExactAlarmPermission() {
    val intent = Intent(Settings.ACTION_REQUEST_SCHEDULE_EXACT_ALARM)
    startActivity(intent)
}
```

---

**Why This Change?**

**Problem:** Apps abusing exact alarms for background work
```
❌ Using exact alarms for:
- Regular data sync (should use WorkManager)
- Frequent uploads (should use WorkManager)
- Battery drain
```

**Solution:** Require permission
```
✅ Users aware of which apps use exact alarms
✅ Can revoke permission
✅ Better battery life
```

---

**Best Practice:**

```kotlin
// Check permission before scheduling
fun setReminder(time: Long) {
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.S) {
        if (!alarmManager.canScheduleExactAlarms()) {
            // Show dialog explaining why permission needed
            showPermissionRationaleDialog {
                // User agreed - open settings
                requestExactAlarmPermission()
            }
            return
        }
    }
    
    // Permission OK - schedule alarm
    scheduleAlarm(time)
}
```

---

### Q5: Repeating alarm কিভাবে set করবেন? setRepeating() এবং setInexactRepeating() এর পার্থক্য কি?

**Answer:**

**1. setRepeating() - Exact Intervals:**

```kotlin
alarmManager.setRepeating(
    AlarmManager.RTC_WAKEUP,
    triggerTimeMillis,     // First trigger time
    intervalMillis,        // Repeat interval
    pendingIntent
)

// Example: Every day at 8:00 AM
val calendar = Calendar.getInstance().apply {
    set(Calendar.HOUR_OF_DAY, 8)
    set(Calendar.MINUTE, 0)
    set(Calendar.SECOND, 0)
}

alarmManager.setRepeating(
    AlarmManager.RTC_WAKEUP,
    calendar.timeInMillis,
    AlarmManager.INTERVAL_DAY, // 24 hours
    pendingIntent
)
```

**Features:**
- Fires at exact intervals
- More battery drain
- Use for user-facing alarms

---

**2. setInexactRepeating() - Approximate Intervals:**

```kotlin
alarmManager.setInexactRepeating(
    AlarmManager.RTC_WAKEUP,
    triggerTimeMillis,
    intervalMillis,
    pendingIntent
)

// Example: Approximately every 15 minutes
alarmManager.setInexactRepeating(
    AlarmManager.RTC_WAKEUP,
    System.currentTimeMillis(),
    AlarmManager.INTERVAL_FIFTEEN_MINUTES,
    pendingIntent
)
```

**Features:**
- System batches alarms with other apps
- More battery efficient
- May fire a few minutes early/late
- Use for background sync

---

**Comparison:**

| Method | Timing | Battery | Use Case |
|--------|--------|---------|----------|
| **setRepeating()** | Exact | Higher drain 🔋❌ | Medicine reminder (8AM, 2PM, 8PM) |
| **setInexactRepeating()** | Approximate | Lower drain 🔋✅ | Background sync |

---

**Available Intervals:**

```kotlin
AlarmManager.INTERVAL_FIFTEEN_MINUTES  // 15 minutes
AlarmManager.INTERVAL_HALF_HOUR        // 30 minutes
AlarmManager.INTERVAL_HOUR             // 1 hour
AlarmManager.INTERVAL_HALF_DAY         // 12 hours
AlarmManager.INTERVAL_DAY              // 24 hours
```

---

**Complete Example:**

```kotlin
// Medicine reminder - exact times
fun scheduleMedicineReminder() {
    val times = listOf("08:00", "14:00", "20:00")
    
    times.forEach { time ->
        val calendar = getCalendarForTime(time)
        
        alarmManager.setRepeating(
            AlarmManager.RTC_WAKEUP,
            calendar.timeInMillis,
            AlarmManager.INTERVAL_DAY, // Every 24 hours
            pendingIntent
        )
    }
}

// Background sync - approximate
fun scheduleBackgroundSync() {
    alarmManager.setInexactRepeating(
        AlarmManager.RTC_WAKEUP,
        System.currentTimeMillis(),
        AlarmManager.INTERVAL_HOUR, // ~Every 1 hour
        pendingIntent
    )
    // System may batch this with other apps' alarms
    // Better battery life
}
```

---

**Important Note (Android 4.4+):**

Starting from Android 4.4 (API 19), all repeating alarms are inexact by default for battery optimization.

```kotlin
// Android 4.4+: setRepeating() behaves like setInexactRepeating()
// To get exact repeating behavior:
// - Use WorkManager for background tasks
// - Or manually reschedule single exact alarms
```

---

### Q6: AlarmManager এবং WorkManager - কখন কোনটা use করবেন?

**Answer:**

**Quick Decision Matrix:**

| Question | AlarmManager | WorkManager |
|----------|--------------|-------------|
| User expects exact time? | ✅ Yes | ❌ No |
| Battery efficiency important? | ❌ No | ✅ Yes |
| Can be delayed? | ❌ No | ✅ Yes |
| Need constraints (WiFi, charging)? | ❌ No | ✅ Yes |
| Foreground notification? | ❌ No | ✅ Yes (optional) |

---

**Detailed Comparison:**

**AlarmManager ✅:**
```kotlin
// Scenario: Morning alarm at 6:00 AM
// User MUST wake up at 6:00 AM EXACTLY

val calendar = Calendar.getInstance().apply {
    set(Calendar.HOUR_OF_DAY, 6)
    set(Calendar.MINUTE, 0)
}

alarmManager.setExact(
    AlarmManager.RTC_WAKEUP,
    calendar.timeInMillis,
    pendingIntent
)

// Characteristics:
✅ Fires at EXACT time
✅ Wakes device
✅ User expects precision
❌ Battery drain
❌ No constraints support
```

**WorkManager ✅:**
```kotlin
// Scenario: Upload photos to server
// Can wait for WiFi, doesn't need exact time

val constraints = Constraints.Builder()
    .setRequiredNetworkType(NetworkType.UNMETERED) // WiFi
    .setRequiresCharging(true)
    .build()

val uploadWork = OneTimeWorkRequestBuilder<UploadWorker>()
    .setConstraints(constraints)
    .build()

WorkManager.getInstance(context).enqueue(uploadWork)

// Characteristics:
✅ Battery efficient
✅ Supports constraints
✅ Guaranteed execution
✅ Respects Doze mode
❌ Not exact time
❌ May be delayed
```

---

**Real-world Scenarios:**

**Use AlarmManager:**
```
1. ⏰ Alarm clock - 6:00 AM exact
2. 💊 Medicine reminder - 8:00 AM, 2:00 PM, 8:00 PM
3. 📅 Meeting reminder - 2:30 PM today
4. 🕌 Prayer time notification - Exact times
5. ⏲️ Cooking timer - Alert after 15 minutes
```

**Use WorkManager:**
```
1. 📤 Upload photos - When WiFi available
2. 🔄 Sync data - Approximately every hour
3. 🗑️ Clear cache - When charging and idle
4. 📊 Upload analytics - When network available
5. 💾 Backup database - Daily, when conditions met
```

---

**Can You Use Both?**

Yes! Different purposes:

```kotlin
class MyApp {
    // AlarmManager for user-facing alarms
    fun setMorningAlarm(hour: Int, minute: Int) {
        alarmManager.setExact(...) // Exact time
    }
    
    // WorkManager for background work
    fun scheduleDataSync() {
        val work = PeriodicWorkRequestBuilder<SyncWorker>(
            1, TimeUnit.HOURS
        ).build()
        
        WorkManager.getInstance(context).enqueue(work)
    }
}
```

---

**Decision Tree:**

```
Need to schedule something?
│
├─ User expects EXACT time? (alarm, reminder)
│  └─ AlarmManager ✅
│
├─ Background work, can be flexible?
│  └─ WorkManager ✅
│
├─ Need constraints? (WiFi, charging, battery)
│  └─ WorkManager ✅
│
└─ Periodic background task?
   ├─ Exact intervals → AlarmManager
   └─ Approximate → WorkManager ✅
```

---

### Q7: AlarmManager এর alarm device reboot হলে কি হবে? কিভাবে handle করবেন?

**Answer:**

**Problem:**

```
Device reboots
    ↓
All AlarmManager alarms are CANCELLED ❌
    ↓
User's alarms lost!
```

**Solution: BOOT_COMPLETED Receiver**

---

**Implementation:**

**Step 1: Add Permission**

```xml
<!-- AndroidManifest.xml -->
<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />
```

---

**Step 2: Register BroadcastReceiver**

```xml
<!-- AndroidManifest.xml -->
<application>
    <receiver
        android:name=".BootReceiver"
        android:enabled="true"
        android:exported="false">
        <intent-filter>
            <action android:name="android.intent.action.BOOT_COMPLETED" />
            <action android:name="android.intent.action.QUICKBOOT_POWERON" />
        </intent-filter>
    </receiver>
</application>
```

---

**Step 3: Create BootReceiver**

```kotlin
class BootReceiver : BroadcastReceiver() {
    
    override fun onReceive(context: Context, intent: Intent) {
        if (intent.action == Intent.ACTION_BOOT_COMPLETED) {
            // Reschedule all alarms
            rescheduleAlarms(context)
        }
    }
    
    private fun rescheduleAlarms(context: Context) {
        // Load saved alarms from database/preferences
        val alarms = loadSavedAlarms(context)
        
        alarms.forEach { alarm ->
            if (alarm.isActive && alarm.timeInMillis > System.currentTimeMillis()) {
                scheduleAlarm(context, alarm)
            }
        }
        
        Log.d("BootReceiver", "Rescheduled ${alarms.size} alarms")
    }
    
    private fun loadSavedAlarms(context: Context): List<Alarm> {
        // Load from SharedPreferences or Database
        val prefs = context.getSharedPreferences("alarms", Context.MODE_PRIVATE)
        val alarms = mutableListOf<Alarm>()
        
        prefs.all.forEach { (key, value) ->
            if (key.startsWith("alarm_")) {
                val alarm = Gson().fromJson(value as String, Alarm::class.java)
                alarms.add(alarm)
            }
        }
        
        return alarms
    }
    
    private fun scheduleAlarm(context: Context, alarm: Alarm) {
        val alarmManager = context.getSystemService(Context.ALARM_SERVICE) 
            as AlarmManager
        
        val intent = Intent(context, AlarmReceiver::class.java).apply {
            putExtra("alarm_id", alarm.id)
            putExtra("title", alarm.title)
            putExtra("message", alarm.message)
        }
        
        val pendingIntent = PendingIntent.getBroadcast(
            context,
            alarm.id,
            intent,
            PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE
        )
        
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
            alarmManager.setExactAndAllowWhileIdle(
                AlarmManager.RTC_WAKEUP,
                alarm.timeInMillis,
                pendingIntent
            )
        } else {
            alarmManager.setExact(
                AlarmManager.RTC_WAKEUP,
                alarm.timeInMillis,
                pendingIntent
            )
        }
    }
}
```

---

**Step 4: Save Alarms Persistently**

```kotlin
class AlarmManager(private val context: Context) {
    
    private val prefs = context.getSharedPreferences("alarms", Context.MODE_PRIVATE)
    
    fun setAlarm(alarm: Alarm) {
        // 1. Save to persistent storage
        saveAlarm(alarm)
        
        // 2. Schedule with AlarmManager
        scheduleAlarm(alarm)
    }
    
    private fun saveAlarm(alarm: Alarm) {
        val json = Gson().toJson(alarm)
        prefs.edit().putString("alarm_${alarm.id}", json).apply()
    }
    
    fun cancelAlarm(alarmId: Int) {
        // 1. Remove from storage
        prefs.edit().remove("alarm_$alarmId").apply()
        
        // 2. Cancel alarm
        cancelScheduledAlarm(alarmId)
    }
}
```

---

**Complete Flow:**

```
1. User sets alarm
   ↓
2. App saves alarm to database/preferences
   ↓
3. App schedules alarm with AlarmManager
   ↓
4. Device reboots
   ↓
5. Android broadcasts BOOT_COMPLETED
   ↓
6. BootReceiver receives broadcast
   ↓
7. BootReceiver loads saved alarms
   ↓
8. BootReceiver reschedules each alarm
   ↓
9. Alarms work again! ✅
```

---

**Important Notes:**

**1. Persistence is Key:**
```kotlin
// ✅ Good - Save alarms
database.alarmDao().insert(alarm)
// or
prefs.edit().putString("alarm_${alarm.id}", json).apply()

// ❌ Bad - Only in memory
val alarms = mutableListOf<Alarm>()
// Lost after reboot!
```

**2. Filter Old Alarms:**
```kotlin
// Only reschedule future alarms
alarms.filter { it.timeInMillis > System.currentTimeMillis() }
```

**3. Handle Repeating Alarms:**
```kotlin
// Repeating alarms - always reschedule
if (alarm.isRepeating) {
    scheduleRepeatingAlarm(alarm)
}
```

---

## সারসংক্ষেপ (Summary)

### Quick Reference:

**Basic Setup:**
```kotlin
// Get AlarmManager
val alarmManager = getSystemService(Context.ALARM_SERVICE) as AlarmManager

// Create Intent
val intent = Intent(context, AlarmReceiver::class.java)

// Create PendingIntent
val pendingIntent = PendingIntent.getBroadcast(
    context,
    requestCode,
    intent,
    PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE
)

// Schedule alarm
alarmManager.setExact(
    AlarmManager.RTC_WAKEUP,
    triggerTimeMillis,
    pendingIntent
)
```

---

### Key Methods:

```kotlin
// One-time exact alarm
setExact(type, triggerTime, pendingIntent)

// One-time exact (works in Doze)
setExactAndAllowWhileIdle(type, triggerTime, pendingIntent)

// Repeating alarm (exact)
setRepeating(type, triggerTime, interval, pendingIntent)

// Repeating alarm (approximate - battery efficient)
setInexactRepeating(type, triggerTime, interval, pendingIntent)

// Cancel alarm
cancel(pendingIntent)
```

---

### Decision Matrix:

| Need | Use |
|------|-----|
| Alarm clock | AlarmManager + RTC_WAKEUP |
| Reminder | AlarmManager + setExact |
| Medicine reminder | AlarmManager + setRepeating |
| Background sync | WorkManager |
| File upload | WorkManager |
| Cache cleanup | WorkManager |

---

### Best Practices:

```
✅ Use setExactAndAllowWhileIdle() for critical alarms
✅ Check canScheduleExactAlarms() on Android 12+
✅ Save alarms persistently (database/preferences)
✅ Reschedule after BOOT_COMPLETED
✅ Use unique request codes for each alarm
✅ Cancel alarms properly (both alarm and PendingIntent)
❌ Don't use for deferrable background work
❌ Don't schedule too frequent alarms
❌ Don't forget RECEIVE_BOOT_COMPLETED permission
```

---

## 🎯 Key Takeaways

1. **AlarmManager = Exact time execution**
2. **RTC_WAKEUP most common** (wakes device)
3. **Android 12+ requires permission** check
4. **setExactAndAllowWhileIdle** for Doze mode
5. **BOOT_COMPLETED** for rescheduling after reboot
6. **Save alarms persistently** (database/preferences)
7. **Use WorkManager** for deferrable work
8. **Battery consideration** - don't overuse

---

**Happy Coding! 🚀**

---

**Author:** Hasibuzzaman Chowdhury  
**Purpose:** Android AlarmManager Mastery  
**Language:** Bangla (Bengali) with English technical terms
