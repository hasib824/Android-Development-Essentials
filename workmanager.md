# Android WorkManager - সম্পূর্ণ Tutorial (Bangla)

> **Background tasks execute করার modern, reliable এবং battery-efficient উপায়**

---

## 📚 Table of Contents
1. [WorkManager কি?](#1-workmanager-কি)
2. [কেন WorkManager ব্যবহার করবেন?](#2-কেন-workmanager-ব্যবহার-করবেন)
3. [WorkManager vs অন্যান্য Solutions](#3-workmanager-vs-অন্যান্য-solutions)
4. [Core Concepts](#4-core-concepts)
5. [Types of Work](#5-types-of-work)
6. [Constraints (শর্তসমূহ)](#6-constraints-শর্তসমূহ)
7. [Setup এবং Dependencies](#7-setup-এবং-dependencies)
8. [OneTime Work - Real Examples](#8-onetime-work---real-examples)
9. [Periodic Work - Real Examples](#9-periodic-work---real-examples)
10. [Advanced Features](#10-advanced-features)
11. [Best Practices](#11-best-practices)
12. [Interview Questions](#12-interview-questions)

---

## 1. WorkManager কি?

### Definition:

**WorkManager** হলো Android Jetpack এর একটি library যা **deferrable (পরে করা যায়), guaranteed background work** execute করার জন্য ব্যবহৃত হয়।

**সহজ বাংলায়:**  
এমন কাজ যা এখনই করতে হবে না, কিন্তু **অবশ্যই** complete হতে হবে (app বন্ধ থাকলেও, device restart হলেও) - সেগুলোর জন্য WorkManager।

---

### Key Features:

#### 1. **Guaranteed Execution (নিশ্চিত সম্পাদন)**
```
✅ App বন্ধ থাকলেও কাজ হবে
✅ Device restart হলেও কাজ হবে
✅ System kill করলেও পরে আবার চালু হবে
```

#### 2. **Battery Efficient**
```
✅ Doze mode respect করে
✅ App Standby mode এ কাজ করে
✅ Battery optimization মেনে চলে
```

#### 3. **Backward Compatible**
```
✅ API 14+ (Android 4.0+) support করে
✅ নিজে থেকে best API select করে:
   - API 23+: JobScheduler
   - API 14-22: AlarmManager + BroadcastReceiver
```

#### 4. **Flexible Constraints**
```
✅ Network connection চাই
✅ Charging হতে হবে
✅ Battery low না থাকতে হবে
✅ Storage পর্যাপ্ত থাকতে হবে
```

---

## 2. কেন WorkManager ব্যবহার করবেন?

### ✅ Use WorkManager When:

#### 1. **Data Sync করতে হবে**
```
📱 Upload user data to server
📱 Sync offline changes
📱 Backup data to cloud
```

#### 2. **Regular Maintenance Tasks**
```
🔄 Clear cache periodically
🔄 Update database
🔄 Delete old files
```

#### 3. **Upload/Download Operations**
```
⬆️ Upload photos to server
⬇️ Download fresh content
⬆️ Send analytics data
```

#### 4. **Processing Tasks**
```
⚙️ Compress images
⚙️ Process videos
⚙️ Generate reports
```

---

### ❌ Don't Use WorkManager When:

#### 1. **Immediate Execution দরকার**
```java
// ❌ Don't use WorkManager
// User clicked button, show result NOW
button.setOnClickListener {
    // Direct API call with Retrofit/Coroutine
    fetchDataNow()
}
```

#### 2. **Exact Time Execution**
```java
// ❌ Don't use WorkManager for alarms
// User set alarm for 6:00 AM exactly
// Use: AlarmManager instead
```

#### 3. **Foreground Service needed**
```java
// ❌ Don't use WorkManager
// Music player, navigation, download with notification
// Use: Foreground Service instead
```

---

## 3. WorkManager vs অন্যান্য Solutions

### Comparison Table:

| Feature | WorkManager | AlarmManager | JobScheduler | Service | Foreground Service |
|---------|-------------|--------------|--------------|---------|-------------------|
| **Guaranteed execution** | ✅ Yes | ✅ Yes | ❌ No | ❌ No | ✅ Yes |
| **Survives app kill** | ✅ Yes | ✅ Yes | ⚠️ Maybe | ❌ No | ⚠️ Maybe |
| **Battery efficient** | ✅ Yes | ❌ No | ✅ Yes | ❌ No | ❌ No |
| **Constraints support** | ✅ Yes | ❌ No | ✅ Yes | ❌ No | ❌ No |
| **Backward compatible** | ✅ API 14+ | ✅ API 1+ | ❌ API 21+ | ✅ API 1+ | ✅ API 1+ |
| **Chaining tasks** | ✅ Yes | ❌ No | ❌ No | ❌ No | ❌ No |
| **User visible** | ❌ No | ❌ No | ❌ No | ❌ No | ✅ Yes (notification) |
| **Use case** | Deferrable work | Exact time | Background jobs | Quick tasks | User-aware tasks |

---

### Decision Tree:

```
Need background work?
│
├─ Must run at EXACT time? (alarm, reminder)
│  └─ Use: AlarmManager
│
├─ User needs to see it? (music, navigation)
│  └─ Use: Foreground Service
│
├─ Quick task, app in foreground?
│  └─ Use: Coroutines / RxJava
│
└─ Deferrable, guaranteed execution?
   └─ Use: WorkManager ✅
```

---

## 4. Core Concepts

### 4 Main Components:

#### 1. **Worker (কর্মী)**
```kotlin
// আসল কাজটা এখানে হয়
class MyWorker(context: Context, params: WorkerParameters) 
    : Worker(context, params) {
    
    override fun doWork(): Result {
        // Do your work here
        return Result.success()
    }
}
```

#### 2. **WorkRequest (কাজের অনুরোধ)**
```kotlin
// OneTimeWorkRequest (একবার)
val oneTimeRequest = OneTimeWorkRequestBuilder<MyWorker>()
    .build()

// PeriodicWorkRequest (বারবার)
val periodicRequest = PeriodicWorkRequestBuilder<MyWorker>(
    repeatInterval = 15, 
    repeatIntervalTimeUnit = TimeUnit.MINUTES
).build()
```

#### 3. **WorkManager (ম্যানেজার)**
```kotlin
// Work enqueue করা (লাইনে দাঁড় করানো)
WorkManager.getInstance(context).enqueue(oneTimeRequest)
```

#### 4. **WorkInfo (তথ্য)**
```kotlin
// Work এর status চেক করা
WorkManager.getInstance(context)
    .getWorkInfoByIdLiveData(oneTimeRequest.id)
    .observe(this) { workInfo ->
        if (workInfo.state == WorkInfo.State.SUCCEEDED) {
            // Work completed!
        }
    }
```

---

### Worker Lifecycle:

```
ENQUEUED → RUNNING → SUCCEEDED
                   → FAILED
                   → CANCELLED
                   → BLOCKED (constraints not met)
```

---

## 5. Types of Work

### 1️⃣ OneTime Work (একবারের কাজ)

**Use case:** একবার execute হবে, তারপর শেষ

```kotlin
// Worker class
class UploadWorker(context: Context, params: WorkerParameters) 
    : Worker(context, params) {
    
    override fun doWork(): Result {
        // Upload file to server
        return try {
            uploadFile()
            Result.success()
        } catch (e: Exception) {
            Result.failure()
        }
    }
    
    private fun uploadFile() {
        // Upload logic here
    }
}

// Create and enqueue work request
val uploadRequest = OneTimeWorkRequestBuilder<UploadWorker>()
    .build()
    
WorkManager.getInstance(context).enqueue(uploadRequest)
```

---

### 2️⃣ Periodic Work (পুনরাবৃত্তিমূলক কাজ)

**Use case:** নির্দিষ্ট সময় পরপর execute হবে

```kotlin
// Worker class
class SyncWorker(context: Context, params: WorkerParameters) 
    : Worker(context, params) {
    
    override fun doWork(): Result {
        // Sync data with server
        syncData()
        return Result.success()
    }
    
    private fun syncData() {
        // Sync logic here
    }
}

// Create and enqueue periodic work request (minimum 15 minutes interval)
val syncRequest = PeriodicWorkRequestBuilder<SyncWorker>(
    15, TimeUnit.MINUTES  // প্রতি 15 মিনিট পরপর
).build()

WorkManager.getInstance(context).enqueue(syncRequest)
```

**⚠️ Important:** Minimum interval = **15 minutes**

---

### 3️⃣ Expedited Work (জরুরি কাজ) - Android 12+

**Use case:** দ্রুত execute হওয়া দরকার কিন্তু Foreground Service না

```kotlin
// Worker class
class UrgentWorker(context: Context, params: WorkerParameters) 
    : Worker(context, params) {
    
    override fun doWork(): Result {
        // Urgent task
        performUrgentTask()
        return Result.success()
    }
    
    private fun performUrgentTask() {
        // Urgent logic here
    }
}

// Create and enqueue expedited work request
val urgentRequest = OneTimeWorkRequestBuilder<UrgentWorker>()
    .setExpedited(OutOfQuotaPolicy.RUN_AS_NON_EXPEDITED_WORK_REQUEST)
    .build()

WorkManager.getInstance(context).enqueue(urgentRequest)
```

---

## 6. Constraints (শর্তসমূহ)

### কাজ কখন হবে তা নিয়ন্ত্রণ করা

```kotlin
// Worker class
class MyWorker(context: Context, params: WorkerParameters) 
    : Worker(context, params) {
    
    override fun doWork(): Result {
        // Your work here - will only run when constraints are met
        performTask()
        return Result.success()
    }
    
    private fun performTask() {
        // Task logic
    }
}

// Define constraints
val constraints = Constraints.Builder()
    .setRequiredNetworkType(NetworkType.CONNECTED)     // Internet লাগবে
    .setRequiresCharging(true)                         // Charging হতে হবে
    .setRequiresBatteryNotLow(true)                    // Battery low না
    .setRequiresStorageNotLow(true)                    // Storage যথেষ্ট
    .setRequiresDeviceIdle(true)                       // Device idle
    .build()

// Apply constraints to work request
val workRequest = OneTimeWorkRequestBuilder<MyWorker>()
    .setConstraints(constraints)
    .build()

// Enqueue
WorkManager.getInstance(context).enqueue(workRequest)
```

---

### Available Constraints:

#### 1. **Network Constraints**

```kotlin
// কোন network লাগবে না
.setRequiredNetworkType(NetworkType.NOT_REQUIRED)

// যেকোনো connection (WiFi/Mobile data)
.setRequiredNetworkType(NetworkType.CONNECTED)

// শুধু WiFi
.setRequiredNetworkType(NetworkType.UNMETERED)

// Internet + not roaming
.setRequiredNetworkType(NetworkType.NOT_ROAMING)

// Metered network (mobile data)
.setRequiredNetworkType(NetworkType.METERED)
```

**Real Example:**
```kotlin
// Large file upload - শুধু WiFi তে
val constraints = Constraints.Builder()
    .setRequiredNetworkType(NetworkType.UNMETERED)  // WiFi only
    .build()

val uploadRequest = OneTimeWorkRequestBuilder<UploadWorker>()
    .setConstraints(constraints)
    .build()
```

---

#### 2. **Battery Constraints**

```kotlin
// Charging হতে হবে
.setRequiresCharging(true)

// Battery low না থাকতে হবে
.setRequiresBatteryNotLow(true)
```

**Real Example:**
```kotlin
// Video processing - battery drain হবে
val constraints = Constraints.Builder()
    .setRequiresCharging(true)           // Charging এ করবে
    .setRequiresBatteryNotLow(true)      // Battery যথেষ্ট থাকতে হবে
    .build()

val processRequest = OneTimeWorkRequestBuilder<VideoProcessWorker>()
    .setConstraints(constraints)
    .build()
```

---

#### 3. **Storage Constraints**

```kotlin
// Storage পর্যাপ্ত থাকতে হবে
.setRequiresStorageNotLow(true)
```

**Real Example:**
```kotlin
// Database cleanup - storage free করবে
val constraints = Constraints.Builder()
    .setRequiresStorageNotLow(false)  // Low storage এ run করবে
    .build()
```

---

#### 4. **Device State Constraints**

```kotlin
// Device idle (ব্যবহার হচ্ছে না) থাকতে হবে
.setRequiresDeviceIdle(true)  // API 23+
```

**Real Example:**
```kotlin
// Heavy maintenance task
val constraints = Constraints.Builder()
    .setRequiresDeviceIdle(true)      // User ব্যবহার করছে না
    .setRequiresCharging(true)        // এবং charging হচ্ছে
    .build()

val maintenanceRequest = OneTimeWorkRequestBuilder<MaintenanceWorker>()
    .setConstraints(constraints)
    .build()
```

---

## 7. Setup এবং Dependencies

### Step 1: Add Dependencies

```gradle
// app/build.gradle.kts
dependencies {
    // WorkManager
    val work_version = "2.9.0"
    
    implementation("androidx.work:work-runtime-ktx:$work_version")
    
    // Optional - RxJava3 support
    implementation("androidx.work:work-rxjava3:$work_version")
    
    // Optional - Test helpers
    androidTestImplementation("androidx.work:work-testing:$work_version")
}
```

---

### Step 2: No Manifest Changes Needed!

WorkManager automatically initializes। কোনো manifest entry লাগবে না।

**কিন্তু custom configuration চাইলে:**

```xml
<!-- AndroidManifest.xml -->
<application>
    <provider
        android:name="androidx.startup.InitializationProvider"
        android:authorities="${applicationId}.androidx-startup"
        android:exported="false"
        tools:node="merge">
        <meta-data
            android:name="androidx.work.WorkManagerInitializer"
            android:value="androidx.startup"
            tools:node="remove" />
    </provider>
</application>
```

```kotlin
// Application class
class MyApplication : Application(), Configuration.Provider {
    override fun getWorkManagerConfiguration() =
        Configuration.Builder()
            .setMinimumLoggingLevel(Log.DEBUG)
            .build()
}
```

---

## 8. OneTime Work - Real Examples

### Example 1: Image Upload to Server

**Scenario:** User একটা photo তুলেছে। এটা server এ upload করতে হবে। Internet না থাকলে পরে upload হবে।

```kotlin
// 1. Worker Class
class ImageUploadWorker(
    context: Context,
    params: WorkerParameters
) : CoroutineWorker(context, params) {
    
    override suspend fun doWork(): Result {
        return try {
            // Get image path from input data
            val imagePath = inputData.getString(KEY_IMAGE_PATH) ?: return Result.failure()
            
            // Upload image
            val response = uploadImage(imagePath)
            
            if (response.isSuccessful) {
                // Output data for next worker or UI
                val outputData = workDataOf(KEY_IMAGE_URL to response.imageUrl)
                Result.success(outputData)
            } else {
                Result.retry()  // Retry করবে
            }
        } catch (e: Exception) {
            Log.e("ImageUpload", "Upload failed", e)
            Result.failure()
        }
    }
    
    private suspend fun uploadImage(path: String): UploadResponse {
        // Retrofit API call
        return apiService.uploadImage(File(path))
    }
    
    companion object {
        const val KEY_IMAGE_PATH = "image_path"
        const val KEY_IMAGE_URL = "image_url"
    }
}

// 2. Enqueue Work
fun uploadImage(imagePath: String) {
    // Input data
    val inputData = workDataOf(
        ImageUploadWorker.KEY_IMAGE_PATH to imagePath
    )
    
    // Constraints - Internet লাগবে
    val constraints = Constraints.Builder()
        .setRequiredNetworkType(NetworkType.CONNECTED)
        .build()
    
    // Work request
    val uploadRequest = OneTimeWorkRequestBuilder<ImageUploadWorker>()
        .setInputData(inputData)
        .setConstraints(constraints)
        .setBackoffCriteria(
            BackoffPolicy.EXPONENTIAL,
            OneTimeWorkRequest.MIN_BACKOFF_MILLIS,
            TimeUnit.MILLISECONDS
        )
        .build()
    
    // Enqueue
    WorkManager.getInstance(applicationContext)
        .enqueueUniqueWork(
            "image_upload_${System.currentTimeMillis()}",
            ExistingWorkPolicy.KEEP,
            uploadRequest
        )
    
    // Observe status (optional)
    WorkManager.getInstance(applicationContext)
        .getWorkInfoByIdLiveData(uploadRequest.id)
        .observe(lifecycleOwner) { workInfo ->
            when (workInfo.state) {
                WorkInfo.State.SUCCEEDED -> {
                    val imageUrl = workInfo.outputData.getString(
                        ImageUploadWorker.KEY_IMAGE_URL
                    )
                    Toast.makeText(context, "Upload successful: $imageUrl", Toast.LENGTH_SHORT).show()
                }
                WorkInfo.State.FAILED -> {
                    Toast.makeText(context, "Upload failed", Toast.LENGTH_SHORT).show()
                }
                WorkInfo.State.RUNNING -> {
                    // Show progress
                }
                else -> {}
            }
        }
}
```

---

### Example 2: Database Cleanup

**Scenario:** Old data delete করতে হবে। Device charging এ থাকলে করবে।

```kotlin
// 1. Worker
class DatabaseCleanupWorker(
    context: Context,
    params: WorkerParameters
) : CoroutineWorker(context, params) {
    
    override suspend fun doWork(): Result {
        return try {
            val database = AppDatabase.getInstance(applicationContext)
            
            // Delete data older than 30 days
            val thirtyDaysAgo = System.currentTimeMillis() - (30 * 24 * 60 * 60 * 1000L)
            val deletedCount = database.userDao().deleteOldRecords(thirtyDaysAgo)
            
            Log.d("Cleanup", "Deleted $deletedCount old records")
            
            // Clear cache
            applicationContext.cacheDir.deleteRecursively()
            
            Result.success()
        } catch (e: Exception) {
            Log.e("Cleanup", "Cleanup failed", e)
            Result.failure()
        }
    }
}

// 2. Schedule Cleanup
fun scheduleCleanup() {
    val constraints = Constraints.Builder()
        .setRequiresCharging(true)           // Charging এ
        .setRequiresBatteryNotLow(true)      // Battery low না
        .setRequiresDeviceIdle(true)         // Device idle
        .build()
    
    val cleanupRequest = OneTimeWorkRequestBuilder<DatabaseCleanupWorker>()
        .setConstraints(constraints)
        .build()
    
    WorkManager.getInstance(applicationContext)
        .enqueue(cleanupRequest)
}
```

---

### Example 3: Send Analytics Data

**Scenario:** Analytics data batch করে server এ পাঠাতে হবে। WiFi তে পাঠাবে।

```kotlin
// 1. Worker
class AnalyticsUploadWorker(
    context: Context,
    params: WorkerParameters
) : CoroutineWorker(context, params) {
    
    private val analyticsDb = AnalyticsDatabase.getInstance(applicationContext)
    
    override suspend fun doWork(): Result {
        return try {
            // Get pending events
            val events = analyticsDb.eventDao().getPendingEvents()
            
            if (events.isEmpty()) {
                return Result.success()
            }
            
            // Upload to server
            val response = apiService.uploadEvents(events)
            
            if (response.isSuccessful) {
                // Mark as uploaded
                analyticsDb.eventDao().markAsUploaded(events.map { it.id })
                Result.success()
            } else {
                Result.retry()
            }
        } catch (e: Exception) {
            Log.e("Analytics", "Upload failed", e)
            Result.retry()  // পরে আবার try করবে
        }
    }
}

// 2. Schedule Upload
fun uploadAnalytics() {
    val constraints = Constraints.Builder()
        .setRequiredNetworkType(NetworkType.UNMETERED)  // শুধু WiFi
        .setRequiresBatteryNotLow(true)
        .build()
    
    val uploadRequest = OneTimeWorkRequestBuilder<AnalyticsUploadWorker>()
        .setConstraints(constraints)
        .setInitialDelay(5, TimeUnit.MINUTES)  // 5 মিনিট পরে শুরু
        .build()
    
    WorkManager.getInstance(applicationContext)
        .enqueueUniqueWork(
            "analytics_upload",
            ExistingWorkPolicy.REPLACE,  // আগেরটা cancel করে নতুন
            uploadRequest
        )
}
```

---

### Example 4: Compress and Upload Video

**Scenario:** Video compress করে upload করতে হবে। Charging + WiFi লাগবে।

```kotlin
// 1. Compression Worker
class VideoCompressionWorker(
    context: Context,
    params: WorkerParameters
) : CoroutineWorker(context, params) {
    
    override suspend fun doWork(): Result {
        return try {
            val videoPath = inputData.getString(KEY_VIDEO_PATH) ?: return Result.failure()
            
            // Show progress notification
            setForeground(createForegroundInfo("Compressing video..."))
            
            // Compress video
            val compressedPath = compressVideo(videoPath) { progress ->
                // Update progress
                setProgressAsync(workDataOf(KEY_PROGRESS to progress))
            }
            
            // Output compressed path
            val outputData = workDataOf(KEY_COMPRESSED_PATH to compressedPath)
            Result.success(outputData)
        } catch (e: Exception) {
            Log.e("VideoCompress", "Compression failed", e)
            Result.failure()
        }
    }
    
    private suspend fun compressVideo(
        inputPath: String,
        onProgress: (Int) -> Unit
    ): String {
        // FFmpeg or MediaCodec compression
        // Update progress periodically
        return "compressed_$inputPath"
    }
    
    private fun createForegroundInfo(message: String): ForegroundInfo {
        val notification = NotificationCompat.Builder(applicationContext, CHANNEL_ID)
            .setContentTitle("Processing Video")
            .setContentText(message)
            .setSmallIcon(R.drawable.ic_video)
            .setOngoing(true)
            .build()
        
        return ForegroundInfo(NOTIFICATION_ID, notification)
    }
    
    companion object {
        const val KEY_VIDEO_PATH = "video_path"
        const val KEY_COMPRESSED_PATH = "compressed_path"
        const val KEY_PROGRESS = "progress"
        const val NOTIFICATION_ID = 1001
        const val CHANNEL_ID = "video_processing"
    }
}

// 2. Upload Worker
class VideoUploadWorker(
    context: Context,
    params: WorkerParameters
) : CoroutineWorker(context, params) {
    
    override suspend fun doWork(): Result {
        return try {
            val videoPath = inputData.getString(
                VideoCompressionWorker.KEY_COMPRESSED_PATH
            ) ?: return Result.failure()
            
            setForeground(createForegroundInfo("Uploading video..."))
            
            // Upload
            val response = apiService.uploadVideo(File(videoPath))
            
            if (response.isSuccessful) {
                // Delete compressed file
                File(videoPath).delete()
                Result.success()
            } else {
                Result.retry()
            }
        } catch (e: Exception) {
            Result.failure()
        }
    }
    
    private fun createForegroundInfo(message: String): ForegroundInfo {
        val notification = NotificationCompat.Builder(applicationContext, 
            VideoCompressionWorker.CHANNEL_ID)
            .setContentTitle("Video Upload")
            .setContentText(message)
            .setSmallIcon(R.drawable.ic_upload)
            .setOngoing(true)
            .build()
        
        return ForegroundInfo(VideoCompressionWorker.NOTIFICATION_ID, notification)
    }
}

// 3. Chain Workers
fun compressAndUploadVideo(videoPath: String) {
    val inputData = workDataOf(VideoCompressionWorker.KEY_VIDEO_PATH to videoPath)
    
    // Constraints for compression
    val compressionConstraints = Constraints.Builder()
        .setRequiresCharging(true)
        .setRequiresBatteryNotLow(true)
        .build()
    
    // Constraints for upload
    val uploadConstraints = Constraints.Builder()
        .setRequiredNetworkType(NetworkType.UNMETERED)  // WiFi only
        .setRequiresCharging(true)
        .build()
    
    // Compression work
    val compressionWork = OneTimeWorkRequestBuilder<VideoCompressionWorker>()
        .setInputData(inputData)
        .setConstraints(compressionConstraints)
        .build()
    
    // Upload work
    val uploadWork = OneTimeWorkRequestBuilder<VideoUploadWorker>()
        .setConstraints(uploadConstraints)
        .build()
    
    // Chain: Compress → Upload
    WorkManager.getInstance(applicationContext)
        .beginWith(compressionWork)
        .then(uploadWork)
        .enqueue()
    
    // Observe progress
    WorkManager.getInstance(applicationContext)
        .getWorkInfoByIdLiveData(compressionWork.id)
        .observe(lifecycleOwner) { workInfo ->
            if (workInfo.state == WorkInfo.State.RUNNING) {
                val progress = workInfo.progress.getInt(
                    VideoCompressionWorker.KEY_PROGRESS, 0
                )
                updateUI(progress)
            }
        }
}
```

---

## 9. Periodic Work - Real Examples

### Example 1: Data Sync Every Hour

**Scenario:** Server থেকে latest data fetch করতে হবে প্রতি ঘণ্টায়।

```kotlin
// 1. Sync Worker
class DataSyncWorker(
    context: Context,
    params: WorkerParameters
) : CoroutineWorker(context, params) {
    
    private val database = AppDatabase.getInstance(applicationContext)
    
    override suspend fun doWork(): Result {
        return try {
            // Fetch latest data from server
            val response = apiService.getLatestData()
            
            if (response.isSuccessful) {
                // Update local database
                database.dataDao().insertAll(response.data)
                
                // Update last sync time
                PreferencesManager.setLastSyncTime(System.currentTimeMillis())
                
                Log.d("Sync", "Synced ${response.data.size} items")
                Result.success()
            } else {
                Result.retry()
            }
        } catch (e: Exception) {
            Log.e("Sync", "Sync failed", e)
            Result.retry()
        }
    }
}

// 2. Schedule Periodic Sync
fun scheduleDataSync() {
    val constraints = Constraints.Builder()
        .setRequiredNetworkType(NetworkType.CONNECTED)
        .setRequiresBatteryNotLow(true)
        .build()
    
    val syncRequest = PeriodicWorkRequestBuilder<DataSyncWorker>(
        1, TimeUnit.HOURS  // প্রতি 1 ঘণ্টায়
    )
        .setConstraints(constraints)
        .setInitialDelay(15, TimeUnit.MINUTES)  // প্রথমবার 15 মিনিট পরে
        .build()
    
    WorkManager.getInstance(applicationContext)
        .enqueueUniquePeriodicWork(
            "data_sync",
            ExistingPeriodicWorkPolicy.KEEP,  // আগের থাকলে রাখবে
            syncRequest
        )
}

// 3. Cancel Sync (if needed)
fun cancelDataSync() {
    WorkManager.getInstance(applicationContext)
        .cancelUniqueWork("data_sync")
}
```

---

### Example 2: Cache Cleanup Every 24 Hours

**Scenario:** প্রতিদিন রাতে cache clear করতে হবে।

```kotlin
// 1. Cache Cleanup Worker
class CacheCleanupWorker(
    context: Context,
    params: WorkerParameters
) : Worker(context, params) {
    
    override fun doWork(): Result {
        return try {
            var totalCleared = 0L
            
            // Clear app cache
            applicationContext.cacheDir.listFiles()?.forEach { file ->
                totalCleared += file.length()
                file.deleteRecursively()
            }
            
            // Clear Glide cache
            Glide.get(applicationContext).clearDiskCache()
            
            // Clear old downloads
            val downloadsDir = applicationContext.getExternalFilesDir(
                Environment.DIRECTORY_DOWNLOADS
            )
            downloadsDir?.listFiles()?.forEach { file ->
                val age = System.currentTimeMillis() - file.lastModified()
                if (age > TimeUnit.DAYS.toMillis(7)) {  // 7 দিনের পুরনো
                    totalCleared += file.length()
                    file.delete()
                }
            }
            
            Log.d("CacheCleanup", "Cleared ${totalCleared / 1024 / 1024} MB")
            
            // Notify user (optional)
            showNotification("Cache cleared: ${totalCleared / 1024 / 1024} MB")
            
            Result.success()
        } catch (e: Exception) {
            Log.e("CacheCleanup", "Cleanup failed", e)
            Result.failure()
        }
    }
    
    private fun showNotification(message: String) {
        val notification = NotificationCompat.Builder(applicationContext, CHANNEL_ID)
            .setContentTitle("Cache Cleanup")
            .setContentText(message)
            .setSmallIcon(R.drawable.ic_clean)
            .build()
        
        NotificationManagerCompat.from(applicationContext)
            .notify(NOTIFICATION_ID, notification)
    }
    
    companion object {
        const val NOTIFICATION_ID = 2001
        const val CHANNEL_ID = "cache_cleanup"
    }
}

// 2. Schedule Cleanup
fun scheduleCacheCleanup() {
    val constraints = Constraints.Builder()
        .setRequiresDeviceIdle(true)      // Device idle তে
        .setRequiresCharging(true)         // Charging এ
        .setRequiresBatteryNotLow(true)
        .build()
    
    val cleanupRequest = PeriodicWorkRequestBuilder<CacheCleanupWorker>(
        24, TimeUnit.HOURS,      // প্রতি 24 ঘণ্টায়
        6, TimeUnit.HOURS        // Flex interval: ±6 hours
    )
        .setConstraints(constraints)
        .build()
    
    WorkManager.getInstance(applicationContext)
        .enqueueUniquePeriodicWork(
            "cache_cleanup",
            ExistingPeriodicWorkPolicy.KEEP,
            cleanupRequest
        )
}
```

---

### Example 3: Location Tracking Every 15 Minutes

**Scenario:** User এর location track করতে হবে নিয়মিত (fitness app)।

```kotlin
// 1. Location Worker
class LocationTrackingWorker(
    context: Context,
    params: WorkerParameters
) : CoroutineWorker(context, params) {
    
    private val fusedLocationClient = LocationServices.getFusedLocationProviderClient(
        applicationContext
    )
    
    override suspend fun doWork(): Result {
        return try {
            // Check permission
            if (!hasLocationPermission()) {
                return Result.failure()
            }
            
            // Get last location
            val location = getLastLocation()
            
            if (location != null) {
                // Save to database
                val locationData = LocationData(
                    latitude = location.latitude,
                    longitude = location.longitude,
                    timestamp = System.currentTimeMillis(),
                    accuracy = location.accuracy
                )
                
                val database = AppDatabase.getInstance(applicationContext)
                database.locationDao().insert(locationData)
                
                // Upload if network available
                if (isNetworkAvailable()) {
                    apiService.uploadLocation(locationData)
                }
                
                Result.success()
            } else {
                Result.retry()
            }
        } catch (e: Exception) {
            Log.e("LocationTracking", "Failed", e)
            Result.retry()
        }
    }
    
    @SuppressLint("MissingPermission")
    private suspend fun getLastLocation(): Location? {
        return suspendCancellableCoroutine { continuation ->
            fusedLocationClient.lastLocation
                .addOnSuccessListener { location ->
                    continuation.resume(location)
                }
                .addOnFailureListener { e ->
                    continuation.resume(null)
                }
        }
    }
    
    private fun hasLocationPermission(): Boolean {
        return ContextCompat.checkSelfPermission(
            applicationContext,
            Manifest.permission.ACCESS_FINE_LOCATION
        ) == PackageManager.PERMISSION_GRANTED
    }
    
    private fun isNetworkAvailable(): Boolean {
        val cm = applicationContext.getSystemService(Context.CONNECTIVITY_SERVICE) 
            as ConnectivityManager
        return cm.activeNetwork != null
    }
}

// 2. Schedule Location Tracking
fun scheduleLocationTracking() {
    val constraints = Constraints.Builder()
        .setRequiresBatteryNotLow(true)
        .build()
    
    val locationRequest = PeriodicWorkRequestBuilder<LocationTrackingWorker>(
        15, TimeUnit.MINUTES  // Minimum interval
    )
        .setConstraints(constraints)
        .build()
    
    WorkManager.getInstance(applicationContext)
        .enqueueUniquePeriodicWork(
            "location_tracking",
            ExistingPeriodicWorkPolicy.KEEP,
            locationRequest
        )
}

// 3. Stop Tracking
fun stopLocationTracking() {
    WorkManager.getInstance(applicationContext)
        .cancelUniqueWork("location_tracking")
}
```

---

### Example 4: News Feed Refresh Every 30 Minutes

**Scenario:** News app এ fresh content load করতে হবে regularly।

```kotlin
// 1. News Refresh Worker
class NewsRefreshWorker(
    context: Context,
    params: WorkerParameters
) : CoroutineWorker(context, params) {
    
    private val newsDatabase = NewsDatabase.getInstance(applicationContext)
    
    override suspend fun doWork(): Result {
        return try {
            // Fetch latest news
            val response = newsApiService.getLatestNews(
                category = inputData.getString(KEY_CATEGORY) ?: "general",
                page = 1
            )
            
            if (response.isSuccessful && response.articles.isNotEmpty()) {
                // Save to database
                newsDatabase.articleDao().insertAll(response.articles)
                
                // Remove old articles (keep only 100)
                newsDatabase.articleDao().deleteOldArticles(100)
                
                // Show notification if new articles
                val newCount = response.articles.size
                if (newCount > 0) {
                    showNewArticlesNotification(newCount)
                }
                
                Result.success()
            } else {
                Result.retry()
            }
        } catch (e: Exception) {
            Log.e("NewsRefresh", "Refresh failed", e)
            Result.retry()
        }
    }
    
    private fun showNewArticlesNotification(count: Int) {
        val intent = Intent(applicationContext, MainActivity::class.java)
        val pendingIntent = PendingIntent.getActivity(
            applicationContext, 0, intent,
            PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE
        )
        
        val notification = NotificationCompat.Builder(applicationContext, CHANNEL_ID)
            .setContentTitle("New Articles")
            .setContentText("$count new articles available")
            .setSmallIcon(R.drawable.ic_news)
            .setContentIntent(pendingIntent)
            .setAutoCancel(true)
            .build()
        
        NotificationManagerCompat.from(applicationContext)
            .notify(NOTIFICATION_ID, notification)
    }
    
    companion object {
        const val KEY_CATEGORY = "category"
        const val NOTIFICATION_ID = 3001
        const val CHANNEL_ID = "news_updates"
    }
}

// 2. Schedule for Different Categories
fun scheduleNewsRefresh(category: String = "general") {
    val inputData = workDataOf(NewsRefreshWorker.KEY_CATEGORY to category)
    
    val constraints = Constraints.Builder()
        .setRequiredNetworkType(NetworkType.CONNECTED)
        .build()
    
    val refreshRequest = PeriodicWorkRequestBuilder<NewsRefreshWorker>(
        30, TimeUnit.MINUTES,    // প্রতি 30 মিনিট
        15, TimeUnit.MINUTES     // Flex: ±15 minutes
    )
        .setInputData(inputData)
        .setConstraints(constraints)
        .build()
    
    WorkManager.getInstance(applicationContext)
        .enqueueUniquePeriodicWork(
            "news_refresh_$category",
            ExistingPeriodicWorkPolicy.REPLACE,
            refreshRequest
        )
}

// 3. Refresh Multiple Categories
fun scheduleAllNewsRefresh() {
    val categories = listOf("general", "technology", "sports", "business")
    
    categories.forEach { category ->
        scheduleNewsRefresh(category)
    }
}
```

---

## 10. Advanced Features

### 1️⃣ Work Chaining (কাজ একের পর এক)

**Scenario:** Image download → Compress → Upload

```kotlin
// Workers
class DownloadImageWorker(...) : CoroutineWorker(...) {
    override suspend fun doWork(): Result {
        val imageUrl = inputData.getString(KEY_URL)!!
        val localPath = downloadImage(imageUrl)
        
        val outputData = workDataOf(KEY_LOCAL_PATH to localPath)
        return Result.success(outputData)
    }
}

class CompressImageWorker(...) : CoroutineWorker(...) {
    override suspend fun doWork(): Result {
        val localPath = inputData.getString(KEY_LOCAL_PATH)!!
        val compressedPath = compressImage(localPath)
        
        val outputData = workDataOf(KEY_COMPRESSED_PATH to compressedPath)
        return Result.success(outputData)
    }
}

class UploadImageWorker(...) : CoroutineWorker(...) {
    override suspend fun doWork(): Result {
        val compressedPath = inputData.getString(KEY_COMPRESSED_PATH)!!
        uploadImage(compressedPath)
        
        return Result.success()
    }
}

// Chain the workers
fun processImage(imageUrl: String) {
    val inputData = workDataOf(DownloadImageWorker.KEY_URL to imageUrl)
    
    val downloadWork = OneTimeWorkRequestBuilder<DownloadImageWorker>()
        .setInputData(inputData)
        .build()
    
    val compressWork = OneTimeWorkRequestBuilder<CompressImageWorker>()
        .build()
    
    val uploadWork = OneTimeWorkRequestBuilder<UploadImageWorker>()
        .build()
    
    // Sequential chain
    WorkManager.getInstance(context)
        .beginWith(downloadWork)      // First
        .then(compressWork)            // Second
        .then(uploadWork)              // Third
        .enqueue()
}
```

---

### 2️⃣ Parallel Work (একসাথে অনেক কাজ)

```kotlin
fun uploadMultipleImages(imagePaths: List<String>) {
    // Create work requests for each image
    val uploadWorks = imagePaths.map { path ->
        val inputData = workDataOf(KEY_IMAGE_PATH to path)
        OneTimeWorkRequestBuilder<ImageUploadWorker>()
            .setInputData(inputData)
            .build()
    }
    
    // Run all in parallel, then do something
    val notificationWork = OneTimeWorkRequestBuilder<NotificationWorker>()
        .build()
    
    WorkManager.getInstance(context)
        .beginWith(uploadWorks)       // All parallel
        .then(notificationWork)       // After all complete
        .enqueue()
}
```

---

### 3️⃣ Unique Work (ডুপ্লিকেট avoid)

```kotlin
// Scenario: একই সময়ে একই কাজ একবারই হবে

// Policy 1: KEEP (পুরনো রাখবে)
WorkManager.getInstance(context)
    .enqueueUniqueWork(
        "sync_data",
        ExistingWorkPolicy.KEEP,  // পুরনো চলমান থাকলে নতুন cancel
        syncWork
    )

// Policy 2: REPLACE (নতুন দিয়ে পুরনো replace)
WorkManager.getInstance(context)
    .enqueueUniqueWork(
        "sync_data",
        ExistingWorkPolicy.REPLACE,  // পুরনো cancel, নতুন start
        syncWork
    )

// Policy 3: APPEND (নতুনটা লাইনে দাঁড়াবে)
WorkManager.getInstance(context)
    .enqueueUniqueWork(
        "sync_data",
        ExistingWorkPolicy.APPEND,  // পুরনো শেষে নতুন
        syncWork
    )
```

---

### 4️⃣ Retry and Backoff (পুনঃপ্রচেষ্টা)

```kotlin
val uploadWork = OneTimeWorkRequestBuilder<UploadWorker>()
    .setBackoffCriteria(
        BackoffPolicy.EXPONENTIAL,      // Exponential backoff
        WorkRequest.MIN_BACKOFF_MILLIS, // Minimum: 10 seconds
        TimeUnit.MILLISECONDS
    )
    .build()

// Retry strategy:
// 1st retry: after 10 seconds
// 2nd retry: after 20 seconds
// 3rd retry: after 40 seconds
// ... exponentially increases
```

**Linear Backoff:**
```kotlin
.setBackoffCriteria(
    BackoffPolicy.LINEAR,
    15,
    TimeUnit.SECONDS
)
// 1st retry: 15s
// 2nd retry: 30s (15 + 15)
// 3rd retry: 45s (15 + 15 + 15)
```

---

### 5️⃣ Progress Updates

```kotlin
class DownloadWorker(...) : CoroutineWorker(...) {
    override suspend fun doWork(): Result {
        val fileUrl = inputData.getString(KEY_URL)!!
        
        downloadFile(fileUrl) { bytesDownloaded, totalBytes ->
            val progress = (bytesDownloaded * 100 / totalBytes).toInt()
            
            // Update progress
            setProgressAsync(workDataOf(KEY_PROGRESS to progress))
        }
        
        return Result.success()
    }
}

// Observe progress
WorkManager.getInstance(context)
    .getWorkInfoByIdLiveData(downloadWork.id)
    .observe(this) { workInfo ->
        if (workInfo.state == WorkInfo.State.RUNNING) {
            val progress = workInfo.progress.getInt(KEY_PROGRESS, 0)
            progressBar.progress = progress
            progressText.text = "$progress%"
        }
    }
```

---

### 6️⃣ Cancel Work

```kotlin
// Cancel specific work
WorkManager.getInstance(context)
    .cancelWorkById(workRequest.id)

// Cancel by unique name
WorkManager.getInstance(context)
    .cancelUniqueWork("image_upload")

// Cancel by tag
WorkManager.getInstance(context)
    .cancelAllWorkByTag("uploads")

// Cancel all work
WorkManager.getInstance(context)
    .cancelAllWork()
```

---

## 11. Best Practices

### ✅ DO

#### 1. **Use Constraints Wisely**

```kotlin
// ✅ Good - Specific constraints
val constraints = Constraints.Builder()
    .setRequiredNetworkType(NetworkType.UNMETERED)  // WiFi for large files
    .setRequiresCharging(true)                       // Heavy task
    .build()

// ❌ Bad - Too restrictive
val constraints = Constraints.Builder()
    .setRequiredNetworkType(NetworkType.UNMETERED)
    .setRequiresCharging(true)
    .setRequiresBatteryNotLow(true)
    .setRequiresDeviceIdle(true)
    .setRequiresStorageNotLow(true)
    .build()
// Work may never run!
```

---

#### 2. **Keep Work Short (< 10 minutes)**

```kotlin
// ✅ Good - Short, focused work
class UploadWorker : CoroutineWorker() {
    override suspend fun doWork(): Result {
        uploadSingleFile()  // Quick operation
        return Result.success()
    }
}

// ❌ Bad - Long-running work
class UploadWorker : CoroutineWorker() {
    override suspend fun doWork(): Result {
        for (i in 1..100) {
            uploadFile(i)  // Takes too long!
        }
        return Result.success()
    }
}

// ✅ Better - Break into smaller works
fun uploadMultipleFiles(files: List<File>) {
    val works = files.map { file ->
        OneTimeWorkRequestBuilder<UploadWorker>()
            .setInputData(workDataOf(KEY_FILE to file.path))
            .build()
    }
    WorkManager.getInstance(context).enqueue(works)
}
```

---

#### 3. **Use Unique Work Names**

```kotlin
// ✅ Good - Prevents duplicates
WorkManager.getInstance(context)
    .enqueueUniqueWork(
        "user_data_sync_${userId}",  // Unique per user
        ExistingWorkPolicy.REPLACE,
        syncWork
    )

// ❌ Bad - Generic name
WorkManager.getInstance(context)
    .enqueue(syncWork)  // Multiple instances can run
```

---

#### 4. **Handle Errors Properly**

```kotlin
// ✅ Good - Proper error handling
override suspend fun doWork(): Result {
    return try {
        performTask()
        Result.success()
    } catch (e: IOException) {
        Result.retry()  // Network error - retry later
    } catch (e: IllegalStateException) {
        Result.failure()  // Logic error - don't retry
    }
}

// ❌ Bad - Catching everything
override suspend fun doWork(): Result {
    return try {
        performTask()
        Result.success()
    } catch (e: Exception) {
        Result.retry()  // May retry forever on logic errors!
    }
}
```

---

#### 5. **Use Tags for Grouping**

```kotlin
// Add tags
val uploadWork = OneTimeWorkRequestBuilder<UploadWorker>()
    .addTag("uploads")
    .addTag("user_${userId}")
    .addTag("media")
    .build()

// Cancel by tag
WorkManager.getInstance(context)
    .cancelAllWorkByTag("uploads")  // Cancel all uploads

// Query by tag
WorkManager.getInstance(context)
    .getWorkInfosByTagLiveData("uploads")
    .observe(this) { workInfos ->
        val activeUploads = workInfos.count { 
            it.state == WorkInfo.State.RUNNING 
        }
    }
```

---

#### 6. **Use CoroutineWorker for Suspend Functions**

```kotlin
// ✅ Good - Use CoroutineWorker for coroutines
class ApiWorker(context: Context, params: WorkerParameters) 
    : CoroutineWorker(context, params) {
    
    override suspend fun doWork(): Result {
        val response = apiService.fetchData()  // Suspend function
        return Result.success()
    }
}

// ❌ Bad - Using Worker for suspend functions
class ApiWorker(context: Context, params: WorkerParameters) 
    : Worker(context, params) {
    
    override fun doWork(): Result {
        // Can't call suspend function directly!
        // Need runBlocking or launch
        return Result.success()
    }
}
```

---

### ❌ DON'T

#### 1. **Don't Use for Immediate Tasks**

```kotlin
// ❌ Bad - WorkManager for immediate task
button.setOnClickListener {
    val work = OneTimeWorkRequestBuilder<FetchDataWorker>().build()
    WorkManager.getInstance(context).enqueue(work)
    // User waits... Work may not start immediately!
}

// ✅ Good - Direct coroutine/RxJava
button.setOnClickListener {
    lifecycleScope.launch {
        val data = apiService.fetchData()
        updateUI(data)
    }
}
```

---

#### 2. **Don't Store Large Data in WorkData**

```kotlin
// ❌ Bad - Storing large data
val imageBytes = loadImage()  // 5 MB image
val inputData = workDataOf(KEY_IMAGE to imageBytes)  // Too large!

// ✅ Good - Store path/URI
val imagePath = saveImageToFile(imageBytes)
val inputData = workDataOf(KEY_IMAGE_PATH to imagePath)
```

**WorkData limit: 10 KB**

---

#### 3. **Don't Forget to Clean Up**

```kotlin
// ✅ Good - Cancel when not needed
override fun onDestroy() {
    super.onDestroy()
    WorkManager.getInstance(context)
        .cancelUniqueWork("user_specific_work")
}

// ❌ Bad - Leave work running
// Work keeps running even after user logs out
```

---

#### 4. **Don't Use for Exact Timing**

```kotlin
// ❌ Bad - Expecting exact time
val work = PeriodicWorkRequestBuilder<ReminderWorker>(
    15, TimeUnit.MINUTES
).build()
// May not run exactly at 15 minutes!

// ✅ Good - Use AlarmManager for exact timing
val alarmManager = getSystemService(AlarmManager::class.java)
alarmManager.setExact(...)
```

---

## 12. Interview Questions

### Q1: WorkManager কি এবং কেন ব্যবহার করবেন?

**Answer:**

**Definition:**  
WorkManager হলো Android Jetpack এর একটি library যা **deferrable, guaranteed background work** execute করার জন্য।

**Main Features:**
1. **Guaranteed execution** - App বন্ধ থাকলেও, device restart হলেও কাজ হবে
2. **Battery efficient** - Doze mode, App Standby respect করে
3. **Constraints support** - Network, charging, battery status check করতে পারে
4. **Backward compatible** - API 14+ support

**কখন ব্যবহার করবেন:**
- Data sync করতে হবে
- File upload/download (deferrable)
- Database cleanup
- Cache clearing
- Analytics upload
- Background processing

**কখন ব্যবহার করবেন না:**
- Immediate execution দরকার (use Coroutines)
- Exact time execution (use AlarmManager)
- User-visible tasks (use Foreground Service)

**Example:**
```kotlin
// Upload image - internet লাগবে
val constraints = Constraints.Builder()
    .setRequiredNetworkType(NetworkType.CONNECTED)
    .build()

val uploadWork = OneTimeWorkRequestBuilder<UploadWorker>()
    .setConstraints(constraints)
    .build()

WorkManager.getInstance(context).enqueue(uploadWork)
```

---

### Q2: OneTime Work এবং Periodic Work এর পার্থক্য কি?

**Answer:**

| Aspect | OneTime Work | Periodic Work |
|--------|--------------|---------------|
| **Execution** | একবার | বারবার (নির্দিষ্ট interval এ) |
| **Class** | `OneTimeWorkRequest` | `PeriodicWorkRequest` |
| **Use case** | Image upload, data sync (once) | Regular sync, cache cleanup |
| **Minimum interval** | N/A | 15 minutes |
| **Chaining** | Yes | No (can't chain periodic) |

**OneTime Example:**
```kotlin
val uploadWork = OneTimeWorkRequestBuilder<UploadWorker>()
    .build()
WorkManager.getInstance(context).enqueue(uploadWork)
```

**Periodic Example:**
```kotlin
val syncWork = PeriodicWorkRequestBuilder<SyncWorker>(
    1, TimeUnit.HOURS  // প্রতি 1 ঘণ্টায়
).build()
WorkManager.getInstance(context).enqueue(syncWork)
```

**Key Points:**
- Periodic work minimum interval: **15 minutes**
- Periodic work can't be chained
- OneTime work can be chained (beginWith → then → then)
- Periodic work uses flex interval for battery optimization

---

### Q3: WorkManager constraints কি কি available আছে?

**Answer:**

**All Available Constraints:**

**1. Network Constraints:**
```kotlin
.setRequiredNetworkType(NetworkType.CONNECTED)      // Any network
.setRequiredNetworkType(NetworkType.UNMETERED)      // WiFi only
.setRequiredNetworkType(NetworkType.NOT_ROAMING)    // Not roaming
.setRequiredNetworkType(NetworkType.METERED)        // Mobile data
.setRequiredNetworkType(NetworkType.NOT_REQUIRED)   // No network needed
```

**2. Battery Constraints:**
```kotlin
.setRequiresCharging(true)         // Must be charging
.setRequiresBatteryNotLow(true)    // Battery not low
```

**3. Storage Constraint:**
```kotlin
.setRequiresStorageNotLow(true)    // Sufficient storage
```

**4. Device State:**
```kotlin
.setRequiresDeviceIdle(true)       // Device idle (API 23+)
```

**Real Example:**
```kotlin
// Video upload - WiFi + Charging
val constraints = Constraints.Builder()
    .setRequiredNetworkType(NetworkType.UNMETERED)  // WiFi only
    .setRequiresCharging(true)                       // While charging
    .setRequiresBatteryNotLow(true)                  // Battery OK
    .build()

val uploadWork = OneTimeWorkRequestBuilder<VideoUploadWorker>()
    .setConstraints(constraints)
    .build()
```

**Constraint Combinations:**

```kotlin
// Light task - Just internet
Constraints.Builder()
    .setRequiredNetworkType(NetworkType.CONNECTED)
    .build()

// Heavy task - WiFi + Charging + Idle
Constraints.Builder()
    .setRequiredNetworkType(NetworkType.UNMETERED)
    .setRequiresCharging(true)
    .setRequiresDeviceIdle(true)
    .build()

// Emergency task - No constraints
Constraints.Builder()
    .build()  // Empty - runs ASAP
```

---

### Q4: Work এর different states কি কি?

**Answer:**

**WorkInfo States:**

```
ENQUEUED → RUNNING → SUCCEEDED
                   → FAILED
                   → CANCELLED
                   
BLOCKED (constraints not met)
```

**1. ENQUEUED:**
```
- Work লাইনে আছে
- Constraints check হচ্ছে
- এখনো start হয়নি
```

**2. RUNNING:**
```
- Work execute হচ্ছে
- doWork() method চলছে
```

**3. SUCCEEDED:**
```
- Work successfully complete হয়েছে
- Result.success() returned
- Final state (can't change)
```

**4. FAILED:**
```
- Work failed
- Result.failure() returned
- No more retries
- Final state
```

**5. CANCELLED:**
```
- Work manually cancelled
- WorkManager.cancelWorkById()
- Final state
```

**6. BLOCKED:**
```
- Constraints not met yet
- Waiting for conditions (WiFi, charging, etc.)
- Not executing
```

**Example - Observing States:**
```kotlin
WorkManager.getInstance(context)
    .getWorkInfoByIdLiveData(workRequest.id)
    .observe(this) { workInfo ->
        when (workInfo.state) {
            WorkInfo.State.ENQUEUED -> {
                showMessage("Work queued")
            }
            WorkInfo.State.RUNNING -> {
                showProgress("Processing...")
            }
            WorkInfo.State.SUCCEEDED -> {
                showMessage("Upload complete")
                hideProgress()
            }
            WorkInfo.State.FAILED -> {
                showError("Upload failed")
                hideProgress()
            }
            WorkInfo.State.BLOCKED -> {
                showMessage("Waiting for WiFi...")
            }
            WorkInfo.State.CANCELLED -> {
                showMessage("Upload cancelled")
            }
        }
    }
```

**State Transitions:**

```
User enqueues work
       ↓
   ENQUEUED (checking constraints)
       ↓
   [Constraints met?]
   Yes ↓         No → BLOCKED (wait for constraints)
   RUNNING              ↓
       ↓           [Constraints met later?]
   doWork()            Yes → RUNNING
       ↓
   [Result?]
   success → SUCCEEDED (final)
   failure → FAILED (final)
   retry → ENQUEUED (try again)
   
   [User cancels?]
       ↓
   CANCELLED (final)
```

---

### Q5: WorkManager কিভাবে internally কাজ করে? কোন API use করে?

**Answer:**

**WorkManager Architecture:**

WorkManager **automatically** select করে best API based on Android version:

**API Selection:**

| Android Version | API Used |
|----------------|----------|
| API 23+ (Android 6.0+) | **JobScheduler** |
| API 14-22 (Android 4.0-5.1) | **AlarmManager + BroadcastReceiver** |

**Internal Components:**

```
┌─────────────────────────────────────┐
│         WorkManager                 │
│  (Public API - Your code uses this) │
└──────────────┬──────────────────────┘
               │
               ↓
┌─────────────────────────────────────┐
│       WorkManager Database          │
│   (SQLite - stores work requests)   │
└──────────────┬──────────────────────┘
               │
               ↓
┌─────────────────────────────────────┐
│         Scheduler                   │
│   (Selects JobScheduler/Alarm)      │
└──────────────┬──────────────────────┘
               │
       ┌───────┴───────┐
       ↓               ↓
┌────────────┐  ┌─────────────┐
│JobScheduler│  │AlarmManager │
│ (API 23+)  │  │ (API 14-22) │
└────────────┘  └─────────────┘
```

**How it Works:**

**1. Enqueue:**
```kotlin
WorkManager.getInstance(context).enqueue(workRequest)
// ↓
// Saves to database
// Creates unique work ID
// Stores constraints, input data
```

**2. Schedule:**
```
Database ← Work saved
    ↓
Scheduler checks Android version
    ↓
API 23+? → Use JobScheduler
API 14-22? → Use AlarmManager + BroadcastReceiver
```

**3. Execute:**
```
Conditions met? (constraints satisfied)
    ↓ Yes
Start Worker
    ↓
doWork() executes
    ↓
Return Result
    ↓
Update database
    ↓
Notify observers (LiveData)
```

**4. Retry (if needed):**
```
Result.retry()?
    ↓
Apply backoff policy
    ↓
Re-enqueue work
    ↓
Schedule again
```

**Database Schema (Simplified):**
```sql
WorkSpec table:
- id (unique work ID)
- state (ENQUEUED, RUNNING, etc.)
- worker_class_name
- input_data (blob)
- output_data (blob)
- constraints
- initial_delay
- backoff_policy
- run_attempt_count
```

**Example - Under the Hood:**

```kotlin
// Your code:
val work = OneTimeWorkRequestBuilder<UploadWorker>().build()
WorkManager.getInstance(context).enqueue(work)

// What happens internally:
// 1. WorkManager creates WorkSpec
// 2. Saves to database:
//    id: "abc-123"
//    state: ENQUEUED
//    worker: UploadWorker
//    constraints: [network: CONNECTED]
//
// 3. Scheduler (API 23+):
//    JobScheduler.schedule(
//        jobId: workId.hashCode(),
//        requirements: NetworkType.ANY
//    )
//
// 4. When conditions met:
//    SystemJobService receives callback
//    Starts UploadWorker.doWork()
//
// 5. After execution:
//    Updates database: state = SUCCEEDED
//    Notifies LiveData observers
```

**Why This Architecture?**
- ✅ **Backwards compatible** - Supports old Android versions
- ✅ **Battery efficient** - Uses system schedulers
- ✅ **Persistent** - Database survives app kill
- ✅ **Reliable** - System manages execution

---

### Q6: Work chaining কি এবং কিভাবে করবেন?

**Answer:**

**Work Chaining = একের পর এক work execute করা**

**Types:**

**1. Sequential Chaining (পর্যায়ক্রমে):**
```kotlin
WorkManager.getInstance(context)
    .beginWith(workA)      // First
    .then(workB)           // Second (after A completes)
    .then(workC)           // Third (after B completes)
    .enqueue()
```

**2. Parallel Chaining (একসাথে):**
```kotlin
WorkManager.getInstance(context)
    .beginWith(listOf(workA, workB, workC))  // All parallel
    .then(workD)                              // After all complete
    .enqueue()
```

**3. Complex Chaining:**
```kotlin
val chain1 = WorkManager.getInstance(context)
    .beginWith(workA)
    .then(workB)

val chain2 = WorkManager.getInstance(context)
    .beginWith(workC)
    .then(workD)

// Combine chains
WorkContinuation.combine(listOf(chain1, chain2))
    .then(workE)  // After both chains complete
    .enqueue()
```

**Real Example - Image Processing:**

```kotlin
// Scenario: Download → Compress → Upload → Notify

// 1. Define Workers
class DownloadWorker : CoroutineWorker() {
    override suspend fun doWork(): Result {
        val url = inputData.getString(KEY_URL)!!
        val path = downloadImage(url)
        
        val output = workDataOf(KEY_PATH to path)
        return Result.success(output)
    }
}

class CompressWorker : CoroutineWorker() {
    override suspend fun doWork(): Result {
        val path = inputData.getString(KEY_PATH)!!
        val compressed = compressImage(path)
        
        val output = workDataOf(KEY_PATH to compressed)
        return Result.success(output)
    }
}

class UploadWorker : CoroutineWorker() {
    override suspend fun doWork(): Result {
        val path = inputData.getString(KEY_PATH)!!
        val url = uploadImage(path)
        
        val output = workDataOf(KEY_URL to url)
        return Result.success(output)
    }
}

class NotifyWorker : CoroutineWorker() {
    override suspend fun doWork(): Result {
        val url = inputData.getString(KEY_URL)!!
        showNotification("Upload complete: $url")
        return Result.success()
    }
}

// 2. Chain Them
fun processAndUploadImage(imageUrl: String) {
    val input = workDataOf(DownloadWorker.KEY_URL to imageUrl)
    
    val downloadWork = OneTimeWorkRequestBuilder<DownloadWorker>()
        .setInputData(input)
        .build()
    
    val compressWork = OneTimeWorkRequestBuilder<CompressWorker>()
        .build()
    
    val uploadWork = OneTimeWorkRequestBuilder<UploadWorker>()
        .setConstraints(
            Constraints.Builder()
                .setRequiredNetworkType(NetworkType.CONNECTED)
                .build()
        )
        .build()
    
    val notifyWork = OneTimeWorkRequestBuilder<NotifyWorker>()
        .build()
    
    // Chain: Download → Compress → Upload → Notify
    WorkManager.getInstance(context)
        .beginWith(downloadWork)
        .then(compressWork)
        .then(uploadWork)
        .then(notifyWork)
        .enqueue()
    
    // Observe final result
    WorkManager.getInstance(context)
        .getWorkInfoByIdLiveData(notifyWork.id)
        .observe(lifecycleOwner) { workInfo ->
            if (workInfo.state.isFinished) {
                Log.d("Chain", "All work completed")
            }
        }
}
```

**Data Passing Between Workers:**

```kotlin
// Worker A
override suspend fun doWork(): Result {
    val result = doSomething()
    
    // Pass to next worker
    val output = workDataOf("key" to result)
    return Result.success(output)
}

// Worker B (receives data from A)
override suspend fun doWork(): Result {
    val input = inputData.getString("key")  // Data from Worker A
    processInput(input)
    return Result.success()
}
```

**Important Rules:**
- ✅ OneTime work can be chained
- ❌ Periodic work cannot be chained
- ✅ If any work fails, chain stops
- ✅ Data flows from one worker to next via outputData → inputData

---

### Q7: WorkManager এবং Foreground Service এর মধ্যে পার্থক্য কি?

**Answer:**

| Feature | WorkManager | Foreground Service |
|---------|-------------|-------------------|
| **User visibility** | Hidden (no notification) | Visible (notification required) |
| **Guaranteed execution** | Yes (even after app kill) | Yes (while running) |
| **Battery impact** | Low (battery optimized) | High (continuous) |
| **Use case** | Deferrable background work | User-aware tasks |
| **Examples** | Data sync, upload, cleanup | Music player, navigation, download |
| **Can be deferred** | Yes | No |
| **Constraints** | Supports (WiFi, charging, etc.) | No built-in constraints |
| **Process priority** | Low | High (harder to kill) |

**When to Use:**

**WorkManager ✅:**
```kotlin
// Upload photos (can wait for WiFi)
// Clear cache (can wait for charging)
// Sync database (can wait for network)
// Send analytics (not urgent)
```

**Foreground Service ✅:**
```kotlin
// Music playback (user is listening NOW)
// GPS navigation (user is navigating NOW)
// File download with progress (user watching)
// Video recording (user recording NOW)
```

**Example Comparison:**

**WorkManager - Image Upload:**
```kotlin
// User takes photo
// Upload can happen later (when WiFi available)
val uploadWork = OneTimeWorkRequestBuilder<UploadWorker>()
    .setConstraints(
        Constraints.Builder()
            .setRequiredNetworkType(NetworkType.UNMETERED)  // Wait for WiFi
            .build()
    )
    .build()

WorkManager.getInstance(context).enqueue(uploadWork)
// No notification
// User doesn't see it
// Happens in background when conditions are met
```

**Foreground Service - Music Player:**
```kotlin
// User presses play
// Music must play NOW
class MusicService : Service() {
    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        val notification = createNotification()  // MUST show notification
        startForeground(NOTIFICATION_ID, notification)
        
        mediaPlayer.play()  // Plays immediately
        return START_STICKY
    }
}

// User sees persistent notification
// Service has high priority
// Won't be killed easily
```

**Can You Combine Both?**

Yes! Use Foreground Service for immediate task, then WorkManager for follow-up:

```kotlin
// User starts download
class DownloadService : Service() {
    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        startForeground(ID, notification)  // Show progress
        
        downloadFile()  // Download now
        
        // After download, schedule upload via WorkManager
        scheduleUploadWork()
        
        stopSelf()
        return START_NOT_STICKY
    }
}

fun scheduleUploadWork() {
    val uploadWork = OneTimeWorkRequestBuilder<UploadWorker>()
        .setConstraints(...)
        .build()
    WorkManager.getInstance(context).enqueue(uploadWork)
}
```

---

### Q8: PeriodicWorkRequest এর minimum interval কত এবং কেন?

**Answer:**

**Minimum Interval: 15 minutes**

```kotlin
// ✅ Valid
val work = PeriodicWorkRequestBuilder<SyncWorker>(
    15, TimeUnit.MINUTES  // Minimum
).build()

// ❌ Invalid - Less than 15 minutes
val work = PeriodicWorkRequestBuilder<SyncWorker>(
    5, TimeUnit.MINUTES   // Will be changed to 15 minutes!
).build()
```

**কেন 15 Minutes Minimum?**

**1. Battery Optimization:**
```
- Frequent execution drains battery
- 15 minutes allows system to batch tasks
- Reduces CPU wake-ups
```

**2. Doze Mode Compatibility:**
```
- Doze mode limits background work
- 15 minutes aligns with maintenance windows
- Ensures work can run during Doze
```

**3. System Resource Management:**
```
- Too frequent = system overhead
- 15 minutes is practical for most tasks
- Prevents abuse by apps
```

**Flex Interval (For More Control):**

```kotlin
// Work can run within a time window
val work = PeriodicWorkRequestBuilder<SyncWorker>(
    1, TimeUnit.HOURS,       // Repeat every 1 hour
    15, TimeUnit.MINUTES     // Flex: Can run in last 15 min of hour
).build()

// Execution window:
// Hour 1: 00:45 - 01:00 (can run anytime in this window)
// Hour 2: 01:45 - 02:00
// Hour 3: 02:45 - 03:00
```

**Why Use Flex Interval?**
```
✅ Battery savings (system batches work)
✅ Less CPU wake-ups
✅ Better user experience (coalescing)
```

**What If You Need More Frequent Updates?**

**Option 1: Use Foreground Service**
```kotlin
// For continuous updates (e.g., step counter)
class StepCounterService : Service() {
    override fun onStartCommand(...): Int {
        startForeground(ID, notification)
        
        // Update every second
        timer.scheduleAtFixedRate(1000) {
            updateStepCount()
        }
        
        return START_STICKY
    }
}
```

**Option 2: Use AlarmManager (Exact Timing)**
```kotlin
// For precise intervals (not recommended for frequent)
val alarmManager = getSystemService(AlarmManager::class.java)
alarmManager.setRepeating(
    AlarmManager.RTC_WAKEUP,
    triggerTime,
    5 * 60 * 1000,  // 5 minutes
    pendingIntent
)
```

**Option 3: Rethink Design**
```
Ask yourself:
- Does user really need 5-minute updates?
- Can you fetch on-demand instead?
- Can you use push notifications from server?
```

**Best Practice:**
```kotlin
// Use longest acceptable interval
val work = PeriodicWorkRequestBuilder<SyncWorker>(
    1, TimeUnit.HOURS,  // Not 15 minutes if hourly is OK
    15, TimeUnit.MINUTES
).build()

// Better battery life
// Better user experience
```

---

### Q9: Work fail হলে কিভাবে retry করবেন?

**Answer:**

**3 Ways to Handle Failures:**

**1. Return Result.retry():**
```kotlin
class UploadWorker : CoroutineWorker() {
    override suspend fun doWork(): Result {
        return try {
            uploadFile()
            Result.success()
        } catch (e: IOException) {
            // Network error - retry later
            Result.retry()
        } catch (e: IllegalStateException) {
            // Logic error - don't retry
            Result.failure()
        }
    }
}
```

**2. Set Backoff Policy:**

**Exponential Backoff (Recommended):**
```kotlin
val work = OneTimeWorkRequestBuilder<UploadWorker>()
    .setBackoffCriteria(
        BackoffPolicy.EXPONENTIAL,
        WorkRequest.MIN_BACKOFF_MILLIS,  // 10 seconds
        TimeUnit.MILLISECONDS
    )
    .build()

// Retry schedule:
// 1st retry: after 10 seconds
// 2nd retry: after 20 seconds (10 * 2^1)
// 3rd retry: after 40 seconds (10 * 2^2)
// 4th retry: after 80 seconds (10 * 2^3)
// ... exponentially increases up to 5 hours max
```

**Linear Backoff:**
```kotlin
val work = OneTimeWorkRequestBuilder<UploadWorker>()
    .setBackoffCriteria(
        BackoffPolicy.LINEAR,
        30,  // 30 seconds
        TimeUnit.SECONDS
    )
    .build()

// Retry schedule:
// 1st retry: after 30 seconds
// 2nd retry: after 60 seconds (30 * 2)
// 3rd retry: after 90 seconds (30 * 3)
// 4th retry: after 120 seconds (30 * 4)
```

**3. Custom Retry Logic:**
```kotlin
class SmartUploadWorker : CoroutineWorker() {
    override suspend fun doWork(): Result {
        val attemptCount = runAttemptCount  // Built-in counter
        
        return try {
            uploadFile()
            Result.success()
        } catch (e: Exception) {
            when {
                attemptCount >= 5 -> {
                    // Too many attempts - give up
                    Log.e("Upload", "Failed after 5 attempts")
                    Result.failure()
                }
                e is IOException && !isNetworkAvailable() -> {
                    // No network - retry later
                    Result.retry()
                }
                e is HttpException && e.code() == 429 -> {
                    // Rate limited - retry with longer delay
                    Result.retry()
                }
                else -> {
                    // Other errors - fail
                    Result.failure()
                }
            }
        }
    }
}
```

**Complete Example with Retry Handling:**

```kotlin
class RobustUploadWorker(
    context: Context,
    params: WorkerParameters
) : CoroutineWorker(context, params) {
    
    override suspend fun doWork(): Result {
        val filePath = inputData.getString(KEY_FILE_PATH) ?: return Result.failure()
        val attemptNumber = runAttemptCount
        
        Log.d("Upload", "Attempt #$attemptNumber for $filePath")
        
        return try {
            // Try upload
            val response = apiService.uploadFile(File(filePath))
            
            if (response.isSuccessful) {
                // Success!
                Log.d("Upload", "Upload successful on attempt #$attemptNumber")
                
                // Clean up file
                File(filePath).delete()
                
                Result.success()
            } else {
                handleFailure(response.code(), attemptNumber)
            }
        } catch (e: IOException) {
            // Network error
            Log.e("Upload", "Network error on attempt #$attemptNumber", e)
            
            if (attemptNumber < 3) {
                Result.retry()  // Retry up to 3 times
            } else {
                Result.failure()  // Give up after 3 attempts
            }
        } catch (e: Exception) {
            // Other errors - don't retry
            Log.e("Upload", "Fatal error", e)
            Result.failure()
        }
    }
    
    private fun handleFailure(code: Int, attemptNumber: Int): Result {
        return when (code) {
            429 -> {
                // Rate limited - always retry
                Log.w("Upload", "Rate limited, will retry")
                Result.retry()
            }
            in 500..599 -> {
                // Server error - retry up to 5 times
                if (attemptNumber < 5) {
                    Result.retry()
                } else {
                    Result.failure()
                }
            }
            else -> {
                // Client error (4xx) - don't retry
                Log.e("Upload", "Client error: $code")
                Result.failure()
            }
        }
    }
    
    companion object {
        const val KEY_FILE_PATH = "file_path"
    }
}

// Schedule with backoff
fun scheduleUpload(filePath: String) {
    val inputData = workDataOf(
        RobustUploadWorker.KEY_FILE_PATH to filePath
    )
    
    val uploadWork = OneTimeWorkRequestBuilder<RobustUploadWorker>()
        .setInputData(inputData)
        .setConstraints(
            Constraints.Builder()
                .setRequiredNetworkType(NetworkType.CONNECTED)
                .build()
        )
        .setBackoffCriteria(
            BackoffPolicy.EXPONENTIAL,
            30,  // Start with 30 seconds
            TimeUnit.SECONDS
        )
        .build()
    
    WorkManager.getInstance(context)
        .enqueueUniqueWork(
            "upload_$filePath",
            ExistingWorkPolicy.REPLACE,
            uploadWork
        )
}
```

**Key Points:**
- ✅ Use `Result.retry()` for transient errors (network, rate limit)
- ✅ Use `Result.failure()` for permanent errors (bad input, auth error)
- ✅ Set appropriate backoff policy (exponential is usually best)
- ✅ Limit retry attempts to avoid infinite loops
- ✅ Log attempts for debugging

---

### Q10: WorkManager এর threading model কি? কোন thread এ work execute হয়?

**Answer:**

**Threading Model:**

WorkManager automatically manages threads। আপনাকে thread handling করতে হয় না।

**Different Worker Types:**

**1. Worker (Synchronous):**
```kotlin
class MyWorker(context: Context, params: WorkerParameters) 
    : Worker(context, params) {
    
    override fun doWork(): Result {
        // ⚠️ Runs on BACKGROUND THREAD (Executor)
        // Can do blocking operations
        
        Thread.sleep(5000)  // OK - blocking call
        
        return Result.success()
    }
}
```
- Runs on: **Background thread** from Executor
- Can do: Blocking I/O, network calls
- Default executor: Fixed thread pool

---

**2. CoroutineWorker (Suspend Functions):**
```kotlin
class MyCoroutineWorker(context: Context, params: WorkerParameters) 
    : CoroutineWorker(context, params) {
    
    override suspend fun doWork(): Result {
        // ⚠️ Runs on Dispatchers.Default
        // Suspend function - can call other suspend functions
        
        delay(5000)  // OK - suspending call
        val response = apiService.getData()  // OK - suspend function
        
        return Result.success()
    }
}
```
- Runs on: **Dispatchers.Default** (by default)
- Can do: Suspend functions, coroutines
- Non-blocking

---

**3. RxWorker (RxJava):**
```kotlin
class MyRxWorker(context: Context, params: WorkerParameters) 
    : RxWorker(context, params) {
    
    override fun createWork(): Single<Result> {
        // ⚠️ Runs on Schedulers.io() by default
        
        return apiService.getData()
            .map { data ->
                processData(data)
                Result.success()
            }
            .onErrorReturn { Result.failure() }
    }
}
```
- Runs on: **Schedulers.io()** (by default)
- Returns: Single<Result>

---

**4. ListenableWorker (Custom Threading):**
```kotlin
class MyListenableWorker(context: Context, params: WorkerParameters) 
    : ListenableWorker(context, params) {
    
    override fun startWork(): ListenableFuture<Result> {
        // ⚠️ YOU control threading
        
        return CallbackToFutureAdapter.getFuture { completer ->
            myExecutor.execute {
                // Your custom thread
                val result = doWork()
                completer.set(result)
            }
        }
    }
}
```
- Runs on: **Your custom thread**
- Full control

---

**Dispatcher Configuration (CoroutineWorker):**

```kotlin
// Default - Dispatchers.Default
class MyWorker : CoroutineWorker() {
    override suspend fun doWork(): Result {
        // Runs on Default dispatcher
        return Result.success()
    }
}

// Custom Dispatcher
class MyWorker : CoroutineWorker() {
    override suspend fun doWork(): Result {
        return withContext(Dispatchers.IO) {
            // Now runs on IO dispatcher
            val data = database.getData()
            Result.success()
        }
    }
}

// Main Thread (not recommended!)
class MyWorker : CoroutineWorker() {
    override suspend fun doWork(): Result {
        return withContext(Dispatchers.Main) {
            // Runs on Main thread
            // ⚠️ Don't do this! WorkManager is for background work
            updateUI()
            Result.success()
        }
    }
}
```

---

**Custom Executor (Worker):**

```kotlin
// Custom WorkManager Configuration
class MyApplication : Application(), Configuration.Provider {
    
    private val myExecutor = Executors.newFixedThreadPool(4)  // 4 threads
    
    override fun getWorkManagerConfiguration(): Configuration {
        return Configuration.Builder()
            .setExecutor(myExecutor)  // Custom executor
            .setMinimumLoggingLevel(Log.DEBUG)
            .build()
    }
}
```

---

**Comparison:**

| Worker Type | Default Thread | Use Case |
|-------------|----------------|----------|
| **Worker** | Background thread (Executor) | Simple blocking tasks |
| **CoroutineWorker** | Dispatchers.Default | Suspend functions, Kotlin |
| **RxWorker** | Schedulers.io() | RxJava reactive code |
| **ListenableWorker** | Your choice | Custom threading needs |

---

**Best Practices:**

```kotlin
// ✅ Good - CoroutineWorker for modern Kotlin
class DataSyncWorker : CoroutineWorker() {
    override suspend fun doWork(): Result {
        val data = apiService.syncData()  // Suspend call
        database.insert(data)              // Suspend call
        return Result.success()
    }
}

// ✅ Good - Worker for Java/blocking code
class LegacyWorker : Worker() {
    override fun doWork(): Result {
        val data = legacyApi.getData()  // Blocking call
        return Result.success()
    }
}

// ❌ Bad - Don't use Worker with coroutines
class BadWorker : Worker() {
    override fun doWork(): Result {
        runBlocking {  // ❌ Bad! Blocks thread
            apiService.getData()
        }
        return Result.success()
    }
}
```

---

**Key Points:**
- ✅ WorkManager handles threading automatically
- ✅ Use CoroutineWorker for Kotlin coroutines (recommended)
- ✅ Use Worker for blocking Java code
- ✅ doWork() runs on background thread, not main thread
- ❌ Don't do UI operations in doWork()
- ✅ Can customize executor/dispatcher if needed

---

## সারসংক্ষেপ (Summary)

### Quick Reference:

**Core Components:**
```kotlin
// Worker
class MyWorker : Worker() {
    override fun doWork(): Result = Result.success()
}

// OneTime Request
val work = OneTimeWorkRequestBuilder<MyWorker>().build()

// Periodic Request  
val work = PeriodicWorkRequestBuilder<MyWorker>(15, TimeUnit.MINUTES).build()

// Constraints
val constraints = Constraints.Builder()
    .setRequiredNetworkType(NetworkType.CONNECTED)
    .setRequiresCharging(true)
    .build()

// Enqueue
WorkManager.getInstance(context).enqueue(work)
```

---

### When to Use:

| Scenario | Solution |
|----------|----------|
| Deferrable background task | ✅ WorkManager |
| Must run at exact time | ❌ AlarmManager |
| User must see it | ❌ Foreground Service |
| Immediate execution | ❌ Coroutines/RxJava |
| Data sync | ✅ WorkManager |
| File upload/download | ✅ WorkManager |
| Cache cleanup | ✅ WorkManager |

---

### Key Features:

```
✅ Guaranteed execution (app closed, device reboot)
✅ Battery efficient (respects Doze, App Standby)
✅ Flexible constraints (network, charging, battery)
✅ Backward compatible (API 14+)
✅ Work chaining (sequential, parallel)
✅ Retry with backoff
✅ Progress updates
✅ Unique work management
```

---

## 🎯 Key Takeaways

1. **WorkManager = Guaranteed background work**
2. **Use constraints for battery efficiency**
3. **Minimum periodic interval = 15 minutes**
4. **OneTime for single tasks, Periodic for recurring**
5. **Chain works for complex workflows**
6. **CoroutineWorker for Kotlin suspend functions**
7. **Observe work status with LiveData**
8. **Don't use for immediate or exact-time tasks**

---

**Happy Coding! 🚀**

---

**Author:** Hasibuzzaman Chowdhury  
**Purpose:** Android WorkManager Mastery  
**Language:** Bangla (Bengali) with English technical terms
