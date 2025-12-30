# 🎯 Android BroadcastReceiver - সম্পূর্ণ বাংলা Tutorial

## 🤔 BroadcastReceiver কি?

### সহজ ভাষায়:
```
BroadcastReceiver = "Radio যা system/app events শোনে"

Real-Life Analogy:
মনে করো একটা radio station:
- System broadcasts message (like radio signals)
- BroadcastReceiver listens (like radio)
- When message received, take action (play song)

Example:
- Phone রিং হচ্ছে → BroadcastReceiver শুনলো → App react করলো
- Battery low → BroadcastReceiver শুনলো → Show warning
- SMS আসলো → BroadcastReceiver শুনলো → Process message
```

---

## 📊 BroadcastReceiver কেন দরকার?

```
Use Cases:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

1. System Events Listen করতে:
   ✅ Battery low/full
   ✅ Network connected/disconnected
   ✅ Screen on/off
   ✅ Phone boot completed
   ✅ SMS/Call received
   ✅ Airplane mode on/off

2. App-to-App Communication:
   ✅ Send data between apps
   ✅ Notify other apps
   ✅ Trigger actions in other apps

3. Scheduled Tasks:
   ✅ AlarmManager with BroadcastReceiver
   ✅ Periodic notifications
   ✅ Reminder system

4. Custom Events:
   ✅ App-specific broadcasts
   ✅ Local broadcasts (same app)
   ✅ Event bus alternative
```

---

## 📊 BroadcastReceiver Types

```
1. Static (Manifest-registered)
   - Declared in AndroidManifest.xml
   - Always listening (even if app closed)
   - Survives app restart
   - Use for: System events

2. Dynamic (Context-registered)
   - Registered in code (Activity/Service)
   - Listens only when app running
   - Must unregister manually
   - Use for: Temporary listeners

3. Local Broadcast
   - Within same app only
   - More secure
   - More efficient
   - Use for: Internal communication
```

---

## 💻 Example 1: Battery Level Monitoring (Dynamic)

### Complete Implementation:

```kotlin
// BatteryReceiver.kt
class BatteryReceiver : BroadcastReceiver() {
    
    override fun onReceive(context: Context, intent: Intent) {
        // ✅ Called when battery level changes
        
        when (intent.action) {
            Intent.ACTION_BATTERY_CHANGED -> {
                // Get battery info
                val level = intent.getIntExtra(BatteryManager.EXTRA_LEVEL, -1)
                val scale = intent.getIntExtra(BatteryManager.EXTRA_SCALE, -1)
                val percentage = (level / scale.toFloat() * 100).toInt()
                
                val status = intent.getIntExtra(BatteryManager.EXTRA_STATUS, -1)
                val isCharging = status == BatteryManager.BATTERY_STATUS_CHARGING ||
                                status == BatteryManager.BATTERY_STATUS_FULL
                
                val plugged = intent.getIntExtra(BatteryManager.EXTRA_PLUGGED, -1)
                val chargingSource = when (plugged) {
                    BatteryManager.BATTERY_PLUGGED_AC -> "AC Charger"
                    BatteryManager.BATTERY_PLUGGED_USB -> "USB"
                    BatteryManager.BATTERY_PLUGGED_WIRELESS -> "Wireless"
                    else -> "Not Charging"
                }
                
                val temperature = intent.getIntExtra(BatteryManager.EXTRA_TEMPERATURE, 0) / 10.0
                val voltage = intent.getIntExtra(BatteryManager.EXTRA_VOLTAGE, 0)
                
                val health = intent.getIntExtra(BatteryManager.EXTRA_HEALTH, -1)
                val healthStatus = when (health) {
                    BatteryManager.BATTERY_HEALTH_GOOD -> "Good"
                    BatteryManager.BATTERY_HEALTH_OVERHEAT -> "Overheating"
                    BatteryManager.BATTERY_HEALTH_DEAD -> "Dead"
                    BatteryManager.BATTERY_HEALTH_OVER_VOLTAGE -> "Over Voltage"
                    BatteryManager.BATTERY_HEALTH_COLD -> "Too Cold"
                    else -> "Unknown"
                }
                
                Log.d("Battery", "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━")
                Log.d("Battery", "🔋 Battery Status Update")
                Log.d("Battery", "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━")
                Log.d("Battery", "📊 Level: $percentage%")
                Log.d("Battery", "⚡ Charging: $isCharging")
                Log.d("Battery", "🔌 Source: $chargingSource")
                Log.d("Battery", "🌡️ Temperature: $temperature°C")
                Log.d("Battery", "⚡ Voltage: ${voltage}mV")
                Log.d("Battery", "💚 Health: $healthStatus")
                Log.d("Battery", "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━")
                
                // Show notification if battery low
                if (percentage <= 20 && !isCharging) {
                    showLowBatteryNotification(context, percentage)
                }
                
                // Show notification if battery full
                if (percentage == 100 && isCharging) {
                    showFullBatteryNotification(context)
                }
                
                // Broadcast to UI (for live updates)
                val updateIntent = Intent("BATTERY_UPDATE").apply {
                    putExtra("PERCENTAGE", percentage)
                    putExtra("IS_CHARGING", isCharging)
                    putExtra("SOURCE", chargingSource)
                    putExtra("TEMPERATURE", temperature)
                }
                LocalBroadcastManager.getInstance(context).sendBroadcast(updateIntent)
            }
        }
    }
    
    private fun showLowBatteryNotification(context: Context, percentage: Int) {
        val channel = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            NotificationChannel(
                "battery_channel",
                "Battery Alerts",
                NotificationManager.IMPORTANCE_HIGH
            )
        } else null
        
        channel?.let {
            val notificationManager = context.getSystemService(NotificationManager::class.java)
            notificationManager.createNotificationChannel(it)
        }
        
        val notification = NotificationCompat.Builder(context, "battery_channel")
            .setContentTitle("⚠️ Low Battery")
            .setContentText("Battery at $percentage%. Please charge soon!")
            .setSmallIcon(android.R.drawable.ic_dialog_alert)
            .setPriority(NotificationCompat.PRIORITY_HIGH)
            .setAutoCancel(true)
            .build()
        
        val notificationManager = NotificationManagerCompat.from(context)
        if (ActivityCompat.checkSelfPermission(
                context,
                Manifest.permission.POST_NOTIFICATIONS
            ) == PackageManager.PERMISSION_GRANTED
        ) {
            notificationManager.notify(1001, notification)
        }
    }
    
    private fun showFullBatteryNotification(context: Context) {
        val channel = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            NotificationChannel(
                "battery_channel",
                "Battery Alerts",
                NotificationManager.IMPORTANCE_DEFAULT
            )
        } else null
        
        channel?.let {
            val notificationManager = context.getSystemService(NotificationManager::class.java)
            notificationManager.createNotificationChannel(it)
        }
        
        val notification = NotificationCompat.Builder(context, "battery_channel")
            .setContentTitle("✅ Battery Full")
            .setContentText("Battery fully charged. Unplug charger!")
            .setSmallIcon(android.R.drawable.ic_dialog_info)
            .setPriority(NotificationCompat.PRIORITY_DEFAULT)
            .setAutoCancel(true)
            .build()
        
        val notificationManager = NotificationManagerCompat.from(context)
        if (ActivityCompat.checkSelfPermission(
                context,
                Manifest.permission.POST_NOTIFICATIONS
            ) == PackageManager.PERMISSION_GRANTED
        ) {
            notificationManager.notify(1002, notification)
        }
    }
}

// MainActivity.kt
class MainActivity : AppCompatActivity() {
    
    private lateinit var binding: ActivityMainBinding
    
    // ✅ Declare receivers
    private val batteryReceiver = BatteryReceiver()
    private val uiUpdateReceiver = object : BroadcastReceiver() {
        override fun onReceive(context: Context, intent: Intent) {
            // Update UI with battery info
            val percentage = intent.getIntExtra("PERCENTAGE", 0)
            val isCharging = intent.getBooleanExtra("IS_CHARGING", false)
            val source = intent.getStringExtra("SOURCE") ?: ""
            val temperature = intent.getDoubleExtra("TEMPERATURE", 0.0)
            
            binding.tvBatteryLevel.text = "$percentage%"
            binding.tvChargingStatus.text = if (isCharging) "⚡ Charging" else "🔋 Not Charging"
            binding.tvChargingSource.text = "Source: $source"
            binding.tvTemperature.text = "Temperature: $temperature°C"
            
            // Update progress bar
            binding.progressBattery.progress = percentage
            
            // Change color based on level
            val color = when {
                percentage <= 20 -> Color.RED
                percentage <= 50 -> Color.rgb(255, 165, 0) // Orange
                else -> Color.GREEN
            }
            binding.progressBattery.progressTintList = ColorStateList.valueOf(color)
        }
    }
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)
        
        Log.d("MainActivity", "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━")
        Log.d("MainActivity", "📱 MainActivity Created")
        Log.d("MainActivity", "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━")
    }
    
    override fun onResume() {
        super.onResume()
        
        Log.d("MainActivity", "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━")
        Log.d("MainActivity", "▶️ onResume() - Registering receivers")
        Log.d("MainActivity", "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━")
        
        // ✅ Register battery receiver (Dynamic)
        val batteryFilter = IntentFilter(Intent.ACTION_BATTERY_CHANGED)
        registerReceiver(batteryReceiver, batteryFilter)
        Log.d("MainActivity", "✅ Battery receiver registered")
        
        // ✅ Register UI update receiver (Local)
        val uiFilter = IntentFilter("BATTERY_UPDATE")
        LocalBroadcastManager.getInstance(this).registerReceiver(uiUpdateReceiver, uiFilter)
        Log.d("MainActivity", "✅ UI update receiver registered")
    }
    
    override fun onPause() {
        super.onPause()
        
        Log.d("MainActivity", "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━")
        Log.d("MainActivity", "⏸️ onPause() - Unregistering receivers")
        Log.d("MainActivity", "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━")
        
        // ✅ Unregister receivers (IMPORTANT!)
        try {
            unregisterReceiver(batteryReceiver)
            Log.d("MainActivity", "✅ Battery receiver unregistered")
        } catch (e: Exception) {
            Log.e("MainActivity", "❌ Error unregistering battery receiver: ${e.message}")
        }
        
        try {
            LocalBroadcastManager.getInstance(this).unregisterReceiver(uiUpdateReceiver)
            Log.d("MainActivity", "✅ UI update receiver unregistered")
        } catch (e: Exception) {
            Log.e("MainActivity", "❌ Error unregistering UI receiver: ${e.message}")
        }
    }
}

// activity_main.xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp"
    android:gravity="center">
    
    <TextView
        android:id="@+id/tvTitle"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="🔋 Battery Monitor"
        android:textSize="24sp"
        android:textStyle="bold"
        android:gravity="center"
        android:layout_marginBottom="32dp"/>
    
    <ProgressBar
        android:id="@+id/progressBattery"
        style="?android:attr/progressBarStyleHorizontal"
        android:layout_width="match_parent"
        android:layout_height="40dp"
        android:max="100"
        android:progress="50"/>
    
    <TextView
        android:id="@+id/tvBatteryLevel"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="50%"
        android:textSize="48sp"
        android:textStyle="bold"
        android:gravity="center"
        android:layout_marginTop="16dp"/>
    
    <TextView
        android:id="@+id/tvChargingStatus"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="⚡ Charging"
        android:textSize="20sp"
        android:gravity="center"
        android:layout_marginTop="16dp"/>
    
    <TextView
        android:id="@+id/tvChargingSource"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Source: AC Charger"
        android:textSize="16sp"
        android:gravity="center"
        android:layout_marginTop="8dp"/>
    
    <TextView
        android:id="@+id/tvTemperature"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Temperature: 25.0°C"
        android:textSize="16sp"
        android:gravity="center"
        android:layout_marginTop="8dp"/>
    
</LinearLayout>
```

---

## 💻 Example 2: Network Connectivity Monitor (Static + Dynamic)

```kotlin
// NetworkReceiver.kt
class NetworkReceiver : BroadcastReceiver() {
    
    override fun onReceive(context: Context, intent: Intent) {
        Log.d("Network", "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━")
        Log.d("Network", "📡 Network State Changed")
        Log.d("Network", "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━")
        
        val connectivityManager = context.getSystemService(Context.CONNECTIVITY_SERVICE) as ConnectivityManager
        
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
            val network = connectivityManager.activeNetwork
            val capabilities = connectivityManager.getNetworkCapabilities(network)
            
            when {
                capabilities == null -> {
                    Log.d("Network", "❌ No Internet Connection")
                    showNetworkNotification(context, "No Internet", "Please check your connection")
                    broadcastNetworkStatus(context, false, "None")
                }
                capabilities.hasTransport(NetworkCapabilities.TRANSPORT_WIFI) -> {
                    Log.d("Network", "✅ WiFi Connected")
                    val wifiManager = context.applicationContext.getSystemService(Context.WIFI_SERVICE) as WifiManager
                    val wifiInfo = wifiManager.connectionInfo
                    val ssid = wifiInfo.ssid.replace("\"", "")
                    val speed = wifiInfo.linkSpeed
                    
                    Log.d("Network", "📶 SSID: $ssid")
                    Log.d("Network", "⚡ Speed: $speed Mbps")
                    
                    showNetworkNotification(context, "WiFi Connected", "Connected to $ssid")
                    broadcastNetworkStatus(context, true, "WiFi: $ssid")
                }
                capabilities.hasTransport(NetworkCapabilities.TRANSPORT_CELLULAR) -> {
                    Log.d("Network", "✅ Mobile Data Connected")
                    
                    val networkType = when {
                        capabilities.hasCapability(NetworkCapabilities.NET_CAPABILITY_NOT_METERED) -> "Unlimited"
                        else -> "Limited"
                    }
                    
                    Log.d("Network", "📱 Type: $networkType")
                    
                    showNetworkNotification(context, "Mobile Data", "Connected via cellular network")
                    broadcastNetworkStatus(context, true, "Mobile Data")
                }
                capabilities.hasTransport(NetworkCapabilities.TRANSPORT_ETHERNET) -> {
                    Log.d("Network", "✅ Ethernet Connected")
                    showNetworkNotification(context, "Ethernet Connected", "Wired connection active")
                    broadcastNetworkStatus(context, true, "Ethernet")
                }
            }
        } else {
            // Legacy code for older devices
            @Suppress("DEPRECATION")
            val networkInfo = connectivityManager.activeNetworkInfo
            
            if (networkInfo != null && networkInfo.isConnected) {
                val type = when (networkInfo.type) {
                    ConnectivityManager.TYPE_WIFI -> "WiFi"
                    ConnectivityManager.TYPE_MOBILE -> "Mobile Data"
                    ConnectivityManager.TYPE_ETHERNET -> "Ethernet"
                    else -> "Unknown"
                }
                Log.d("Network", "✅ Connected via $type")
                broadcastNetworkStatus(context, true, type)
            } else {
                Log.d("Network", "❌ No Connection")
                broadcastNetworkStatus(context, false, "None")
            }
        }
        
        Log.d("Network", "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━")
    }
    
    private fun showNetworkNotification(context: Context, title: String, message: String) {
        val channel = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            NotificationChannel(
                "network_channel",
                "Network Status",
                NotificationManager.IMPORTANCE_DEFAULT
            )
        } else null
        
        channel?.let {
            val notificationManager = context.getSystemService(NotificationManager::class.java)
            notificationManager.createNotificationChannel(it)
        }
        
        val notification = NotificationCompat.Builder(context, "network_channel")
            .setContentTitle(title)
            .setContentText(message)
            .setSmallIcon(android.R.drawable.ic_dialog_info)
            .setPriority(NotificationCompat.PRIORITY_DEFAULT)
            .setAutoCancel(true)
            .build()
        
        val notificationManager = NotificationManagerCompat.from(context)
        if (ActivityCompat.checkSelfPermission(
                context,
                Manifest.permission.POST_NOTIFICATIONS
            ) == PackageManager.PERMISSION_GRANTED
        ) {
            notificationManager.notify(2001, notification)
        }
    }
    
    private fun broadcastNetworkStatus(context: Context, isConnected: Boolean, type: String) {
        val intent = Intent("NETWORK_STATUS_UPDATE").apply {
            putExtra("IS_CONNECTED", isConnected)
            putExtra("TYPE", type)
        }
        LocalBroadcastManager.getInstance(context).sendBroadcast(intent)
    }
}

// MainActivity.kt
class MainActivity : AppCompatActivity() {
    
    private lateinit var binding: ActivityMainBinding
    private val networkReceiver = NetworkReceiver()
    
    private val networkUpdateReceiver = object : BroadcastReceiver() {
        override fun onReceive(context: Context, intent: Intent) {
            val isConnected = intent.getBooleanExtra("IS_CONNECTED", false)
            val type = intent.getStringExtra("TYPE") ?: "Unknown"
            
            if (isConnected) {
                binding.tvNetworkStatus.text = "✅ Connected"
                binding.tvNetworkType.text = "Type: $type"
                binding.viewStatusIndicator.setBackgroundColor(Color.GREEN)
            } else {
                binding.tvNetworkStatus.text = "❌ No Connection"
                binding.tvNetworkType.text = "Type: None"
                binding.viewStatusIndicator.setBackgroundColor(Color.RED)
            }
        }
    }
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)
    }
    
    override fun onResume() {
        super.onResume()
        
        // ✅ Register network receiver
        val networkFilter = IntentFilter().apply {
            addAction(ConnectivityManager.CONNECTIVITY_ACTION)
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
                addAction(ConnectivityManager.CONNECTIVITY_ACTION)
            }
        }
        registerReceiver(networkReceiver, networkFilter)
        
        // ✅ Register UI update receiver
        val uiFilter = IntentFilter("NETWORK_STATUS_UPDATE")
        LocalBroadcastManager.getInstance(this).registerReceiver(networkUpdateReceiver, uiFilter)
        
        // ✅ Check initial status
        checkNetworkStatus()
    }
    
    override fun onPause() {
        super.onPause()
        
        unregisterReceiver(networkReceiver)
        LocalBroadcastManager.getInstance(this).unregisterReceiver(networkUpdateReceiver)
    }
    
    private fun checkNetworkStatus() {
        val connectivityManager = getSystemService(Context.CONNECTIVITY_SERVICE) as ConnectivityManager
        
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
            val network = connectivityManager.activeNetwork
            val capabilities = connectivityManager.getNetworkCapabilities(network)
            
            val isConnected = capabilities != null
            val type = when {
                capabilities == null -> "None"
                capabilities.hasTransport(NetworkCapabilities.TRANSPORT_WIFI) -> "WiFi"
                capabilities.hasTransport(NetworkCapabilities.TRANSPORT_CELLULAR) -> "Mobile Data"
                capabilities.hasTransport(NetworkCapabilities.TRANSPORT_ETHERNET) -> "Ethernet"
                else -> "Unknown"
            }
            
            updateUI(isConnected, type)
        }
    }
    
    private fun updateUI(isConnected: Boolean, type: String) {
        if (isConnected) {
            binding.tvNetworkStatus.text = "✅ Connected"
            binding.tvNetworkType.text = "Type: $type"
            binding.viewStatusIndicator.setBackgroundColor(Color.GREEN)
        } else {
            binding.tvNetworkStatus.text = "❌ No Connection"
            binding.tvNetworkType.text = "Type: None"
            binding.viewStatusIndicator.setBackgroundColor(Color.RED)
        }
    }
}

// AndroidManifest.xml
<manifest>
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/>
    <uses-permission android:name="android.permission.POST_NOTIFICATIONS"/>
    
    <application>
        <!-- ✅ Static receiver (survives app restart) -->
        <receiver
            android:name=".NetworkReceiver"
            android:enabled="true"
            android:exported="false">
            <intent-filter>
                <action android:name="android.net.conn.CONNECTIVITY_CHANGE"/>
            </intent-filter>
        </receiver>
    </application>
</manifest>
```

---

## 💻 Example 3: Boot Completed Receiver (Static)

```kotlin
// BootReceiver.kt
class BootReceiver : BroadcastReceiver() {
    
    override fun onReceive(context: Context, intent: Intent) {
        
        if (intent.action == Intent.ACTION_BOOT_COMPLETED) {
            Log.d("Boot", "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━")
            Log.d("Boot", "🚀 Device Boot Completed")
            Log.d("Boot", "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━")
            
            // ✅ Start your service after boot
            val serviceIntent = Intent(context, BackgroundService::class.java)
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
                context.startForegroundService(serviceIntent)
            } else {
                context.startService(serviceIntent)
            }
            Log.d("Boot", "✅ Background service started")
            
            // ✅ Schedule alarms
            scheduleAlarms(context)
            
            // ✅ Show notification
            showBootNotification(context)
            
            // ✅ Restore app state
            restoreAppState(context)
        }
    }
    
    private fun scheduleAlarms(context: Context) {
        val alarmManager = context.getSystemService(Context.ALARM_SERVICE) as AlarmManager
        
        // Schedule daily alarm at 9 AM
        val calendar = Calendar.getInstance().apply {
            set(Calendar.HOUR_OF_DAY, 9)
            set(Calendar.MINUTE, 0)
            set(Calendar.SECOND, 0)
            
            if (timeInMillis < System.currentTimeMillis()) {
                add(Calendar.DAY_OF_MONTH, 1)
            }
        }
        
        val intent = Intent(context, AlarmReceiver::class.java)
        val pendingIntent = PendingIntent.getBroadcast(
            context,
            100,
            intent,
            PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE
        )
        
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.S) {
            if (alarmManager.canScheduleExactAlarms()) {
                alarmManager.setRepeating(
                    AlarmManager.RTC_WAKEUP,
                    calendar.timeInMillis,
                    AlarmManager.INTERVAL_DAY,
                    pendingIntent
                )
                Log.d("Boot", "✅ Daily alarm scheduled at 9 AM")
            }
        } else {
            alarmManager.setRepeating(
                AlarmManager.RTC_WAKEUP,
                calendar.timeInMillis,
                AlarmManager.INTERVAL_DAY,
                pendingIntent
            )
            Log.d("Boot", "✅ Daily alarm scheduled at 9 AM")
        }
    }
    
    private fun showBootNotification(context: Context) {
        val channel = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            NotificationChannel(
                "boot_channel",
                "Boot Notifications",
                NotificationManager.IMPORTANCE_DEFAULT
            )
        } else null
        
        channel?.let {
            val notificationManager = context.getSystemService(NotificationManager::class.java)
            notificationManager.createNotificationChannel(it)
        }
        
        val notification = NotificationCompat.Builder(context, "boot_channel")
            .setContentTitle("App Started")
            .setContentText("Background services resumed after boot")
            .setSmallIcon(android.R.drawable.ic_dialog_info)
            .setPriority(NotificationCompat.PRIORITY_DEFAULT)
            .setAutoCancel(true)
            .build()
        
        val notificationManager = NotificationManagerCompat.from(context)
        if (ActivityCompat.checkSelfPermission(
                context,
                Manifest.permission.POST_NOTIFICATIONS
            ) == PackageManager.PERMISSION_GRANTED
        ) {
            notificationManager.notify(3001, notification)
        }
    }
    
    private fun restoreAppState(context: Context) {
        val prefs = context.getSharedPreferences("app_prefs", Context.MODE_PRIVATE)
        
        // Restore settings
        val wasServiceRunning = prefs.getBoolean("service_running", false)
        val lastSyncTime = prefs.getLong("last_sync", 0)
        
        Log.d("Boot", "📋 Restoring app state...")
        Log.d("Boot", "Service was running: $wasServiceRunning")
        Log.d("Boot", "Last sync: ${Date(lastSyncTime)}")
        
        // Check if sync needed
        val timeSinceSync = System.currentTimeMillis() - lastSyncTime
        val hoursSinceSync = timeSinceSync / (1000 * 60 * 60)
        
        if (hoursSinceSync > 24) {
            Log.d("Boot", "⚠️ Sync needed (${hoursSinceSync}h since last sync)")
            // Trigger sync using WorkManager
            scheduleSyncWork(context)
        }
    }
    
    private fun scheduleSyncWork(context: Context) {
        val syncRequest = OneTimeWorkRequestBuilder<SyncWorker>()
            .build()
        
        WorkManager.getInstance(context).enqueue(syncRequest)
        Log.d("Boot", "✅ Sync work scheduled")
    }
}

// AndroidManifest.xml
<manifest>
    <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>
    <uses-permission android:name="android.permission.FOREGROUND_SERVICE"/>
    <uses-permission android:name="android.permission.POST_NOTIFICATIONS"/>
    
    <application>
        <!-- ✅ Boot receiver -->
        <receiver
            android:name=".BootReceiver"
            android:enabled="true"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.BOOT_COMPLETED"/>
                <action android:name="android.intent.action.QUICKBOOT_POWERON"/>
            </intent-filter>
        </receiver>
    </application>
</manifest>
```

---

## 💻 Example 4: SMS Receiver (Static)

```kotlin
// SmsReceiver.kt
class SmsReceiver : BroadcastReceiver() {
    
    companion object {
        const val SMS_RECEIVED = "android.provider.Telephony.SMS_RECEIVED"
    }
    
    override fun onReceive(context: Context, intent: Intent) {
        
        if (intent.action == SMS_RECEIVED) {
            Log.d("SMS", "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━")
            Log.d("SMS", "📨 SMS Received")
            Log.d("SMS", "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━")
            
            val bundle = intent.extras
            if (bundle != null) {
                try {
                    val pdus = bundle.get("pdus") as Array<*>
                    
                    pdus.forEach { pdu ->
                        val message = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
                            val format = bundle.getString("format")
                            SmsMessage.createFromPdu(pdu as ByteArray, format)
                        } else {
                            @Suppress("DEPRECATION")
                            SmsMessage.createFromPdu(pdu as ByteArray)
                        }
                        
                        val sender = message.originatingAddress ?: "Unknown"
                        val messageBody = message.messageBody
                        val timestamp = message.timestampMillis
                        
                        Log.d("SMS", "📱 From: $sender")
                        Log.d("SMS", "💬 Message: $messageBody")
                        Log.d("SMS", "🕐 Time: ${Date(timestamp)}")
                        
                        // ✅ Process SMS (e.g., check for OTP)
                        if (messageBody.contains("OTP", ignoreCase = true)) {
                            val otp = extractOTP(messageBody)
                            Log.d("SMS", "🔐 OTP Detected: $otp")
                            
                            // Broadcast OTP to app
                            val otpIntent = Intent("SMS_OTP_RECEIVED").apply {
                                putExtra("OTP", otp)
                                putExtra("SENDER", sender)
                            }
                            LocalBroadcastManager.getInstance(context).sendBroadcast(otpIntent)
                            
                            // Show notification
                            showOTPNotification(context, otp)
                        }
                        
                        // ✅ Check for specific sender
                        if (sender.contains("BANK", ignoreCase = true)) {
                            Log.d("SMS", "🏦 Bank SMS detected")
                            processBankSMS(context, sender, messageBody)
                        }
                        
                    }
                    
                } catch (e: Exception) {
                    Log.e("SMS", "❌ Error processing SMS: ${e.message}")
                }
            }
            
            Log.d("SMS", "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━")
        }
    }
    
    private fun extractOTP(message: String): String {
        // Extract 4-6 digit OTP from message
        val pattern = "\\d{4,6}".toRegex()
        val match = pattern.find(message)
        return match?.value ?: ""
    }
    
    private fun showOTPNotification(context: Context, otp: String) {
        val channel = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            NotificationChannel(
                "sms_channel",
                "SMS Notifications",
                NotificationManager.IMPORTANCE_HIGH
            )
        } else null
        
        channel?.let {
            val notificationManager = context.getSystemService(NotificationManager::class.java)
            notificationManager.createNotificationChannel(it)
        }
        
        // Create action to copy OTP
        val copyIntent = Intent(context, CopyOTPReceiver::class.java).apply {
            putExtra("OTP", otp)
        }
        val copyPendingIntent = PendingIntent.getBroadcast(
            context,
            0,
            copyIntent,
            PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE
        )
        
        val notification = NotificationCompat.Builder(context, "sms_channel")
            .setContentTitle("🔐 OTP Received")
            .setContentText("Your OTP: $otp")
            .setSmallIcon(android.R.drawable.ic_dialog_info)
            .setPriority(NotificationCompat.PRIORITY_HIGH)
            .addAction(android.R.drawable.ic_menu_upload, "Copy OTP", copyPendingIntent)
            .setAutoCancel(true)
            .build()
        
        val notificationManager = NotificationManagerCompat.from(context)
        if (ActivityCompat.checkSelfPermission(
                context,
                Manifest.permission.POST_NOTIFICATIONS
            ) == PackageManager.PERMISSION_GRANTED
        ) {
            notificationManager.notify(4001, notification)
        }
    }
    
    private fun processBankSMS(context: Context, sender: String, message: String) {
        Log.d("SMS", "🏦 Processing bank SMS...")
        
        // Extract transaction info
        val amountPattern = "Rs\\.?\\s?(\\d+(?:,\\d+)*(?:\\.\\d{2})?)".toRegex()
        val amountMatch = amountPattern.find(message)
        val amount = amountMatch?.groupValues?.get(1)
        
        if (amount != null) {
            Log.d("SMS", "💰 Amount: Rs. $amount")
            
            // Save to database
            saveTransaction(context, sender, amount, message)
            
            // Show notification
            showTransactionNotification(context, amount)
        }
    }
    
    private fun saveTransaction(context: Context, sender: String, amount: String, message: String) {
        val prefs = context.getSharedPreferences("transactions", Context.MODE_PRIVATE)
        val transactionId = System.currentTimeMillis().toString()
        
        prefs.edit().apply {
            putString("txn_$transactionId", "$sender|$amount|$message")
            apply()
        }
        
        Log.d("SMS", "✅ Transaction saved: $transactionId")
    }
    
    private fun showTransactionNotification(context: Context, amount: String) {
        val notification = NotificationCompat.Builder(context, "sms_channel")
            .setContentTitle("💰 Transaction Alert")
            .setContentText("Amount: Rs. $amount")
            .setSmallIcon(android.R.drawable.ic_dialog_info)
            .setPriority(NotificationCompat.PRIORITY_HIGH)
            .setAutoCancel(true)
            .build()
        
        val notificationManager = NotificationManagerCompat.from(context)
        if (ActivityCompat.checkSelfPermission(
                context,
                Manifest.permission.POST_NOTIFICATIONS
            ) == PackageManager.PERMISSION_GRANTED
        ) {
            notificationManager.notify(4002, notification)
        }
    }
}

// CopyOTPReceiver.kt
class CopyOTPReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        val otp = intent.getStringExtra("OTP")
        
        if (otp != null) {
            val clipboard = context.getSystemService(Context.CLIPBOARD_SERVICE) as ClipboardManager
            val clip = ClipData.newPlainText("OTP", otp)
            clipboard.setPrimaryClip(clip)
            
            Toast.makeText(context, "OTP copied: $otp", Toast.LENGTH_SHORT).show()
            Log.d("OTP", "✅ OTP copied to clipboard: $otp")
        }
    }
}

// AndroidManifest.xml
<manifest>
    <uses-permission android:name="android.permission.RECEIVE_SMS"/>
    <uses-permission android:name="android.permission.READ_SMS"/>
    <uses-permission android:name="android.permission.POST_NOTIFICATIONS"/>
    
    <application>
        <!-- ✅ SMS receiver -->
        <receiver
            android:name=".SmsReceiver"
            android:enabled="true"
            android:exported="true">
            <intent-filter android:priority="999">
                <action android:name="android.provider.Telephony.SMS_RECEIVED"/>
            </intent-filter>
        </receiver>
        
        <!-- ✅ Copy OTP receiver -->
        <receiver
            android:name=".CopyOTPReceiver"
            android:enabled="true"
            android:exported="false"/>
    </application>
</manifest>
```

---

## 💻 Example 5: Custom Broadcast (App-to-App Communication)

```kotlin
// Sender App - MainActivity.kt
class SenderActivity : AppCompatActivity() {
    
    private lateinit var binding: ActivitySenderBinding
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        binding = ActivitySenderBinding.inflate(layoutInflater)
        setContentView(binding.root)
        
        binding.btnSendBroadcast.setOnClickListener {
            sendCustomBroadcast()
        }
        
        binding.btnSendLocalBroadcast.setOnClickListener {
            sendLocalBroadcast()
        }
    }
    
    private fun sendCustomBroadcast() {
        val message = binding.etMessage.text.toString()
        
        Log.d("Sender", "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━")
        Log.d("Sender", "📤 Sending Custom Broadcast")
        Log.d("Sender", "Message: $message")
        Log.d("Sender", "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━")
        
        // ✅ Send explicit broadcast (secure)
        val intent = Intent("com.example.CUSTOM_ACTION").apply {
            putExtra("MESSAGE", message)
            putExtra("TIMESTAMP", System.currentTimeMillis())
            putExtra("SENDER", "SenderApp")
            // Make it explicit by setting package
            setPackage("com.example.receiverapp")
        }
        
        sendBroadcast(intent)
        
        Toast.makeText(this, "Broadcast sent!", Toast.LENGTH_SHORT).show()
        Log.d("Sender", "✅ Broadcast sent")
    }
    
    private fun sendLocalBroadcast() {
        val message = binding.etMessage.text.toString()
        
        Log.d("Sender", "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━")
        Log.d("Sender", "📤 Sending Local Broadcast")
        Log.d("Sender", "Message: $message")
        Log.d("Sender", "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━")
        
        // ✅ Local broadcast (same app only)
        val intent = Intent("LOCAL_MESSAGE").apply {
            putExtra("MESSAGE", message)
        }
        
        LocalBroadcastManager.getInstance(this).sendBroadcast(intent)
        
        Toast.makeText(this, "Local broadcast sent!", Toast.LENGTH_SHORT).show()
        Log.d("Sender", "✅ Local broadcast sent")
    }
}

// Receiver App - CustomReceiver.kt
class CustomReceiver : BroadcastReceiver() {
    
    override fun onReceive(context: Context, intent: Intent) {
        
        if (intent.action == "com.example.CUSTOM_ACTION") {
            Log.d("Receiver", "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━")
            Log.d("Receiver", "📥 Custom Broadcast Received")
            Log.d("Receiver", "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━")
            
            val message = intent.getStringExtra("MESSAGE") ?: ""
            val timestamp = intent.getLongExtra("TIMESTAMP", 0)
            val sender = intent.getStringExtra("SENDER") ?: "Unknown"
            
            Log.d("Receiver", "💬 Message: $message")
            Log.d("Receiver", "🕐 Time: ${Date(timestamp)}")
            Log.d("Receiver", "📱 From: $sender")
            Log.d("Receiver", "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━")
            
            // Show notification
            showMessageNotification(context, message, sender)
            
            // Save to database
            saveMessage(context, message, timestamp, sender)
            
            // Broadcast to UI
            val uiIntent = Intent("MESSAGE_RECEIVED").apply {
                putExtra("MESSAGE", message)
                putExtra("SENDER", sender)
            }
            LocalBroadcastManager.getInstance(context).sendBroadcast(uiIntent)
        }
    }
    
    private fun showMessageNotification(context: Context, message: String, sender: String) {
        val channel = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            NotificationChannel(
                "message_channel",
                "Messages",
                NotificationManager.IMPORTANCE_HIGH
            )
        } else null
        
        channel?.let {
            val notificationManager = context.getSystemService(NotificationManager::class.java)
            notificationManager.createNotificationChannel(it)
        }
        
        val notification = NotificationCompat.Builder(context, "message_channel")
            .setContentTitle("Message from $sender")
            .setContentText(message)
            .setSmallIcon(android.R.drawable.ic_dialog_email)
            .setPriority(NotificationCompat.PRIORITY_HIGH)
            .setAutoCancel(true)
            .build()
        
        val notificationManager = NotificationManagerCompat.from(context)
        if (ActivityCompat.checkSelfPermission(
                context,
                Manifest.permission.POST_NOTIFICATIONS
            ) == PackageManager.PERMISSION_GRANTED
        ) {
            notificationManager.notify(5001, notification)
        }
    }
    
    private fun saveMessage(context: Context, message: String, timestamp: Long, sender: String) {
        val prefs = context.getSharedPreferences("messages", Context.MODE_PRIVATE)
        val messageId = timestamp.toString()
        
        prefs.edit().apply {
            putString("msg_$messageId", "$sender|$message|$timestamp")
            apply()
        }
        
        Log.d("Receiver", "✅ Message saved")
    }
}

// Receiver App - AndroidManifest.xml
<manifest package="com.example.receiverapp">
    <uses-permission android:name="android.permission.POST_NOTIFICATIONS"/>
    
    <application>
        <!-- ✅ Custom broadcast receiver -->
        <receiver
            android:name=".CustomReceiver"
            android:enabled="true"
            android:exported="true">
            <intent-filter>
                <action android:name="com.example.CUSTOM_ACTION"/>
            </intent-filter>
        </receiver>
    </application>
</manifest>
```

---

## 📊 BroadcastReceiver Lifecycle

```
BroadcastReceiver Lifecycle:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

1. Broadcast Sent
   ↓
2. System checks registered receivers
   ↓
3. onReceive() called
   ↓
4. Execute your code (max 10 seconds!)
   ↓
5. onReceive() returns
   ↓
6. Receiver destroyed (if not static)

Important Notes:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
⏱️ Time Limit: 10 seconds max in onReceive()
❌ Don't do: Long operations, network calls
✅ Do instead: Start service, schedule WorkManager
⚠️ No UI: Can't show Activity/Dialog directly
✅ Can do: Show notification, start service
```

---

## 📊 Static vs Dynamic Receivers

```
Static (Manifest-registered):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ Works even if app closed
✅ Survives app restart
✅ Survives device reboot (with BOOT_COMPLETED)
❌ Can't unregister (always listening)
❌ Limited in Android 8+ (background execution limits)
✅ Use for: System events, boot events

Example:
<receiver android:name=".MyReceiver">
    <intent-filter>
        <action android:name="android.intent.action.BOOT_COMPLETED"/>
    </intent-filter>
</receiver>

Dynamic (Code-registered):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
❌ Only works when app running
❌ Destroyed when Activity/Service destroyed
✅ Can register/unregister anytime
✅ More control over lifecycle
✅ No background execution limits
✅ Use for: Temporary listeners, UI updates

Example:
val receiver = MyReceiver()
val filter = IntentFilter("MY_ACTION")
registerReceiver(receiver, filter)
// Later:
unregisterReceiver(receiver)

Local Broadcast:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ Same app only (secure)
✅ More efficient (no IPC)
✅ Can't be intercepted by other apps
❌ Deprecated (use LiveData/Flow instead)
✅ Alternative: EventBus, LiveData, StateFlow

Example:
val intent = Intent("MY_ACTION")
LocalBroadcastManager.getInstance(context).sendBroadcast(intent)
```

---

## 💡 Best Practices

```kotlin
✅ DO:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ Keep onReceive() fast (<10 seconds)
✅ Use goAsync() for longer operations
✅ Start Service for heavy work
✅ Use WorkManager for background tasks
✅ Always unregister dynamic receivers
✅ Check permissions before sending
✅ Use explicit intents (set package)
✅ Handle null intents gracefully
✅ Use Local broadcasts for internal communication
✅ Check Android version compatibility

❌ DON'T:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
❌ Do long operations in onReceive()
❌ Make network calls in onReceive()
❌ Show Activity from receiver
❌ Forget to unregister dynamic receivers
❌ Use implicit broadcasts (Android 8+)
❌ Assume broadcast will be delivered immediately
❌ Rely on broadcast order
❌ Store receiver instance in static variable
```

---

## 🎯 goAsync() for Longer Operations

```kotlin
class MyReceiver : BroadcastReceiver() {
    
    override fun onReceive(context: Context, intent: Intent) {
        
        // ✅ Use goAsync() for operations > 10 seconds
        val pendingResult = goAsync()
        
        // Run in background thread
        Thread {
            try {
                // Long operation here (but keep under 1 minute total)
                Thread.sleep(15000)  // 15 seconds
                
                Log.d("Receiver", "Long operation complete")
                
            } finally {
                // ✅ IMPORTANT: Must call finish()!
                pendingResult.finish()
            }
        }.start()
    }
}

/*
goAsync() Details:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
- Extends time limit from 10s to ~1 minute
- Receiver not destroyed immediately
- Must call pendingResult.finish() when done
- Use for: Database operations, file I/O
- Don't use for: Network calls (use Service)
*/
```

---

## 🎓 Summary

```
BroadcastReceiver:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

What:
- Component that listens to system/app broadcasts
- Like a radio receiving signals
- Responds to events

When to Use:
✅ System events (battery, network, boot)
✅ App-to-app communication
✅ Scheduled tasks (with AlarmManager)
✅ SMS/call monitoring
✅ Internal app events

Types:
1. Static: Manifest-registered, always listening
2. Dynamic: Code-registered, temporary
3. Local: Same app only, secure

Lifecycle:
Broadcast sent → onReceive() → Execute → Return → Destroyed

Key Points:
⏱️ Keep onReceive() fast (<10 seconds)
🚀 Use Service for heavy work
📦 Use WorkManager for background tasks
🔒 Use explicit intents for security
📱 Always unregister dynamic receivers
✅ Check permissions before accessing data

Common Use Cases:
🔋 Battery monitoring
📡 Network connectivity
📱 SMS/Call detection
🚀 Boot completed
⏰ Alarm notifications
💬 App-to-app messaging
```

---

**Created by: Claude for Hasibuzzaman Chowdhury**  
**Date: December 31, 2025**

**মনে রাখো: BroadcastReceiver = System events listen করার সবচেয়ে ভালো উপায়! Battery, Network, SMS, Boot - সব monitor করতে পারো! কিন্তু onReceive() তে long operations করো না, Service/WorkManager use করো! 🎯🚀**
