# OkHttp - সম্পূর্ণ Bangla Tutorial

## 📚 Table of Contents
1. [OkHttp কি?](#okhttp-কি)
2. [কেন OkHttp দরকার?](#কেন-okhttp-দরকার)
3. [Key Features](#key-features)
4. [Setup Guide](#setup-guide)
5. [Basic Usage](#basic-usage)
6. [Interceptors](#interceptors)
7. [Advanced Features](#advanced-features)
8. [Real World Examples](#real-world-examples)
9. [Best Practices](#best-practices)
10. [Troubleshooting](#troubleshooting)

---

## OkHttp কি?

**OkHttp** হলো একটা **HTTP client library** যা Android এবং Java applications এ network requests করার জন্য use করা হয়।

### 🎯 সহজ কথায়:
```
OkHttp = ইন্টারনেট থেকে data আনা এবং পাঠানোর tool

আপনার App → OkHttp → Internet → Server
                 ↓
            Response ফিরে আসে
                 ↓
        আপনার App data পায়
```

### 💡 Real Life Example:
```kotlin
মনে করুন আপনি একটা Restaurant App বানাচ্ছেন:

User clicks "Menu দেখাও" button
        ↓
আপনার App OkHttp use করে server থেকে menu request করে
        ↓
Server menu data (JSON) পাঠায়
        ↓
OkHttp response নিয়ে আসে
        ↓
আপনার App menu display করে

এই পুরো network communication টা OkHttp handle করে!
```

---

## কেন OkHttp দরকার?

### 🤔 Problem: Built-in HTTP Client এর সমস্যা

Android এ built-in `HttpURLConnection` আছে, কিন্তু এটা use করা কঠিন:
```kotlin
// ❌ HttpURLConnection - Complex & Difficult

val url = URL("https://api.example.com/users")
val connection = url.openConnection() as HttpURLConnection

try {
    connection.requestMethod = "GET"
    connection.connectTimeout = 5000
    connection.readTimeout = 5000
    
    val responseCode = connection.responseCode
    
    if (responseCode == HttpURLConnection.HTTP_OK) {
        val inputStream = connection.inputStream
        val reader = BufferedReader(InputStreamReader(inputStream))
        val response = StringBuilder()
        var line: String?
        
        while (reader.readLine().also { line = it } != null) {
            response.append(line)
        }
        
        reader.close()
        // Now parse JSON manually...
    }
} catch (e: Exception) {
    e.printStackTrace()
} finally {
    connection.disconnect()
}

// 😱 এত code শুধু একটা GET request এর জন্য!
```

### ✅ Solution: OkHttp - Simple & Powerful
```kotlin
// ✅ OkHttp - Simple & Clean

val client = OkHttpClient()

val request = Request.Builder()
    .url("https://api.example.com/users")
    .build()

client.newCall(request).execute().use { response ->
    if (response.isSuccessful) {
        val data = response.body?.string()
        // Use data!
    }
}

// ✨ মাত্র 10 lines! Clean & Readable!
```

### 🎁 OkHttp এর সুবিধা:

| Feature | Built-in | OkHttp |
|---------|----------|--------|
| **Easy to use** | ❌ Complex | ✅ Simple API |
| **Connection Pooling** | ❌ Manual | ✅ Automatic |
| **Response Caching** | ❌ Manual | ✅ Built-in |
| **GZIP Compression** | ❌ Manual | ✅ Automatic |
| **HTTP/2 Support** | ❌ Limited | ✅ Full Support |
| **Interceptors** | ❌ No | ✅ Powerful |
| **Retry Logic** | ❌ Manual | ✅ Automatic |
| **Thread Safe** | ⚠️ Careful | ✅ Thread-safe |

---

## Key Features

### 1️⃣ Connection Pooling
```kotlin
// OkHttp automatically connection reuse করে

Request 1 → Server (new connection)
Request 2 → Server (reuse same connection) ✅ Fast!
Request 3 → Server (reuse same connection) ✅ Fast!

Benefits:
✅ Faster requests
✅ Less battery usage
✅ Reduced server load
```

### 2️⃣ Automatic Retries
```kotlin
// Network fail হলে automatic retry করে

Your Request
    ↓
Network Failed (timeout)
    ↓
OkHttp: "Let me try again..." 🔄
    ↓
Request Successful! ✅

আপনাকে manually retry logic লিখতে হয় না!
```

### 3️⃣ Response Caching
```kotlin
// Response cache করে রাখে - offline support!

First Request → Server → Cache করে রাখে
Second Request → Cache থেকে দেয় (no internet needed!) ⚡

Benefits:
✅ Faster loading
✅ Offline support
✅ Less data usage
```

### 4️⃣ GZIP Compression
```kotlin
// Automatically data compress করে

Server sends: 1 MB data
    ↓
OkHttp compresses: 200 KB (80% less!) 🗜️
    ↓
Your app receives: 200 KB

Benefits:
✅ Faster downloads
✅ Less data usage
✅ Automatic (transparent)
```

### 5️⃣ Interceptors
```kotlin
// Request/Response modify করতে পারেন

Request → Interceptor 1 (add auth token)
       → Interceptor 2 (add headers)
       → Server

Response ← Interceptor 1 (log response)
        ← Interceptor 2 (check errors)
        ← Your app

Powerful feature! আমরা পরে detail এ দেখব।
```

---

## Setup Guide

### 📋 Step 1: Add Dependencies

#### Using versions.toml:
```toml
# gradle/libs.versions.toml

[versions]
okhttp = "4.12.0"

[libraries]
# OkHttp core
okhttp = { group = "com.squareup.okhttp3", name = "okhttp", version.ref = "okhttp" }

# Logging interceptor (for debugging)
okhttp-logging = { group = "com.squareup.okhttp3", name = "logging-interceptor", version.ref = "okhttp" }
```

#### build.gradle.kts:
```kotlin
dependencies {
    // OkHttp
    implementation(libs.okhttp)
    
    // Logging (optional but recommended for debugging)
    implementation(libs.okhttp.logging)
}
```

### 📋 Step 2: Add Internet Permission
```xml
<!-- AndroidManifest.xml -->

<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    
    <!-- ✅ Internet permission -->
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    
    <application ...>
        ...
    </application>
</manifest>
```

### 📋 Step 3: Setup OkHttpClient (Recommended with Hilt)
```kotlin
// data/di/NetworkModule.kt

import dagger.Module
import dagger.Provides
import dagger.hilt.InstallIn
import dagger.hilt.components.SingletonComponent
import okhttp3.OkHttpClient
import okhttp3.logging.HttpLoggingInterceptor
import java.util.concurrent.TimeUnit
import javax.inject.Singleton

@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {
    
    @Provides
    @Singleton
    fun provideOkHttpClient(): OkHttpClient {
        return OkHttpClient.Builder()
            // Timeouts
            .connectTimeout(30, TimeUnit.SECONDS)
            .readTimeout(30, TimeUnit.SECONDS)
            .writeTimeout(30, TimeUnit.SECONDS)
            
            // Logging (debug only)
            .addInterceptor(
                HttpLoggingInterceptor().apply {
                    level = if (BuildConfig.DEBUG) {
                        HttpLoggingInterceptor.Level.BODY
                    } else {
                        HttpLoggingInterceptor.Level.NONE
                    }
                }
            )
            
            // Retry on connection failure
            .retryOnConnectionFailure(true)
            
            .build()
    }
}
```

---

## Basic Usage

### 1️⃣ Simple GET Request
```kotlin
import okhttp3.OkHttpClient
import okhttp3.Request
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.withContext

class UserRepository @Inject constructor(
    private val client: OkHttpClient
) {
    
    suspend fun getUsers(): Result<String> = withContext(Dispatchers.IO) {
        try {
            // 1. Build request
            val request = Request.Builder()
                .url("https://jsonplaceholder.typicode.com/users")
                .get()
                .build()
            
            // 2. Execute request
            val response = client.newCall(request).execute()
            
            // 3. Check if successful
            if (response.isSuccessful) {
                val body = response.body?.string() ?: ""
                Result.success(body)
            } else {
                Result.failure(Exception("Error: ${response.code}"))
            }
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}
```

### 2️⃣ POST Request with JSON Body
```kotlin
import okhttp3.MediaType.Companion.toMediaType
import okhttp3.RequestBody.Companion.toRequestBody
import org.json.JSONObject

suspend fun createUser(name: String, email: String): Result<String> {
    return withContext(Dispatchers.IO) {
        try {
            // 1. Create JSON body
            val json = JSONObject().apply {
                put("name", name)
                put("email", email)
            }
            
            val mediaType = "application/json; charset=utf-8".toMediaType()
            val requestBody = json.toString().toRequestBody(mediaType)
            
            // 2. Build request
            val request = Request.Builder()
                .url("https://jsonplaceholder.typicode.com/users")
                .post(requestBody)
                .build()
            
            // 3. Execute
            val response = client.newCall(request).execute()
            
            if (response.isSuccessful) {
                Result.success(response.body?.string() ?: "")
            } else {
                Result.failure(Exception("Error: ${response.code}"))
            }
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}
```

### 3️⃣ PUT Request
```kotlin
suspend fun updateUser(userId: String, name: String): Result<String> {
    return withContext(Dispatchers.IO) {
        try {
            val json = JSONObject().apply {
                put("name", name)
            }
            
            val mediaType = "application/json".toMediaType()
            val body = json.toString().toRequestBody(mediaType)
            
            val request = Request.Builder()
                .url("https://jsonplaceholder.typicode.com/users/$userId")
                .put(body)  // ✅ PUT method
                .build()
            
            val response = client.newCall(request).execute()
            
            if (response.isSuccessful) {
                Result.success(response.body?.string() ?: "")
            } else {
                Result.failure(Exception("Error: ${response.code}"))
            }
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}
```

### 4️⃣ DELETE Request
```kotlin
suspend fun deleteUser(userId: String): Result<Unit> {
    return withContext(Dispatchers.IO) {
        try {
            val request = Request.Builder()
                .url("https://jsonplaceholder.typicode.com/users/$userId")
                .delete()  // ✅ DELETE method
                .build()
            
            val response = client.newCall(request).execute()
            
            if (response.isSuccessful) {
                Result.success(Unit)
            } else {
                Result.failure(Exception("Error: ${response.code}"))
            }
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}
```

### 5️⃣ Request with Headers
```kotlin
suspend fun getUserWithAuth(token: String): Result<String> {
    return withContext(Dispatchers.IO) {
        try {
            val request = Request.Builder()
                .url("https://api.example.com/user")
                .header("Authorization", "Bearer $token")  // ✅ Auth header
                .header("Accept", "application/json")
                .header("User-Agent", "MyApp/1.0")
                .build()
            
            val response = client.newCall(request).execute()
            
            if (response.isSuccessful) {
                Result.success(response.body?.string() ?: "")
            } else {
                Result.failure(Exception("Error: ${response.code}"))
            }
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}
```

### 6️⃣ Query Parameters
```kotlin
suspend fun searchUsers(query: String, page: Int): Result<String> {
    return withContext(Dispatchers.IO) {
        try {
            // Build URL with query parameters
            val url = HttpUrl.Builder()
                .scheme("https")
                .host("api.example.com")
                .addPathSegment("users")
                .addQueryParameter("q", query)      // ?q=query
                .addQueryParameter("page", page.toString())  // &page=1
                .addQueryParameter("limit", "20")   // &limit=20
                .build()
            
            val request = Request.Builder()
                .url(url)
                .build()
            
            val response = client.newCall(request).execute()
            
            if (response.isSuccessful) {
                Result.success(response.body?.string() ?: "")
            } else {
                Result.failure(Exception("Error: ${response.code}"))
            }
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}

// Result URL: https://api.example.com/users?q=john&page=1&limit=20
```

---

## Interceptors

**Interceptors** = Request/Response এর মাঝে যা কিছু modify করতে চান তার জন্য powerful tool!

### 🎯 Interceptor কি?
```
Your Request
     ↓
[Interceptor 1] → Modify request (add token)
     ↓
[Interceptor 2] → Log request
     ↓
Network (Server)
     ↓
[Interceptor 2] → Log response
     ↓
[Interceptor 1] → Check errors
     ↓
Your App
```

### 1️⃣ Logging Interceptor (Debugging)
```kotlin
import okhttp3.logging.HttpLoggingInterceptor

val loggingInterceptor = HttpLoggingInterceptor().apply {
    level = if (BuildConfig.DEBUG) {
        HttpLoggingInterceptor.Level.BODY  // Full logs
    } else {
        HttpLoggingInterceptor.Level.NONE  // No logs in production
    }
}

val client = OkHttpClient.Builder()
    .addInterceptor(loggingInterceptor)
    .build()

// Logcat Output:
// --> GET https://api.example.com/users
// --> END GET
// <-- 200 OK (125ms)
// Content-Type: application/json
// {"users": [...]}
// <-- END HTTP
```

### 2️⃣ Authentication Interceptor
```kotlin
class AuthInterceptor(
    private val tokenManager: TokenManager
) : Interceptor {
    
    override fun intercept(chain: Interceptor.Chain): Response {
        // 1. Get original request
        val originalRequest = chain.request()
        
        // 2. Get token
        val token = tokenManager.getToken()
        
        // 3. Add token to request (if exists)
        val newRequest = if (token != null) {
            originalRequest.newBuilder()
                .header("Authorization", "Bearer $token")
                .build()
        } else {
            originalRequest
        }
        
        // 4. Proceed with modified request
        return chain.proceed(newRequest)
    }
}

// Usage:
val client = OkHttpClient.Builder()
    .addInterceptor(AuthInterceptor(tokenManager))
    .build()

// ✨ এখন সব request এ automatic token add হবে!
```

### 3️⃣ Error Handling Interceptor
```kotlin
class ErrorHandlingInterceptor : Interceptor {
    
    override fun intercept(chain: Interceptor.Chain): Response {
        val request = chain.request()
        
        // Execute request
        val response = try {
            chain.proceed(request)
        } catch (e: IOException) {
            // Network error
            throw NetworkException("No internet connection")
        }
        
        // Check response code
        when (response.code) {
            401 -> throw UnauthorizedException("Please login again")
            403 -> throw ForbiddenException("Access denied")
            404 -> throw NotFoundException("Resource not found")
            500 -> throw ServerException("Server error")
            in 200..299 -> return response  // Success
            else -> throw HttpException("Error: ${response.code}")
        }
    }
}

// Custom Exceptions:
class NetworkException(message: String) : IOException(message)
class UnauthorizedException(message: String) : IOException(message)
class ForbiddenException(message: String) : IOException(message)
class NotFoundException(message: String) : IOException(message)
class ServerException(message: String) : IOException(message)
class HttpException(message: String) : IOException(message)
```

### 4️⃣ Retry Interceptor
```kotlin
class RetryInterceptor(
    private val maxRetries: Int = 3
) : Interceptor {
    
    override fun intercept(chain: Interceptor.Chain): Response {
        val request = chain.request()
        var response: Response? = null
        var exception: IOException? = null
        
        // Try multiple times
        repeat(maxRetries) { attempt ->
            try {
                response = chain.proceed(request)
                
                // If successful, return
                if (response!!.isSuccessful) {
                    return response!!
                }
                
                // Close previous response
                response?.close()
                
            } catch (e: IOException) {
                exception = e
                
                // Wait before retry (exponential backoff)
                if (attempt < maxRetries - 1) {
                    Thread.sleep((attempt + 1) * 1000L)
                }
            }
        }
        
        // All retries failed
        throw exception ?: IOException("Request failed after $maxRetries attempts")
    }
}

// Usage:
val client = OkHttpClient.Builder()
    .addInterceptor(RetryInterceptor(maxRetries = 3))
    .build()
```

### 5️⃣ Cache Control Interceptor
```kotlin
class CacheInterceptor(
    private val maxAge: Int = 60  // seconds
) : Interceptor {
    
    override fun intercept(chain: Interceptor.Chain): Response {
        val request = chain.request()
        
        // Add cache control to request
        val newRequest = request.newBuilder()
            .header("Cache-Control", "public, max-age=$maxAge")
            .build()
        
        val response = chain.proceed(newRequest)
        
        // Add cache control to response
        return response.newBuilder()
            .header("Cache-Control", "public, max-age=$maxAge")
            .removeHeader("Pragma")  // Remove old cache header
            .build()
    }
}
```

### 6️⃣ Custom Headers Interceptor
```kotlin
class HeadersInterceptor : Interceptor {
    
    override fun intercept(chain: Interceptor.Chain): Response {
        val request = chain.request()
        
        val newRequest = request.newBuilder()
            .header("Accept", "application/json")
            .header("Content-Type", "application/json")
            .header("User-Agent", "MyApp/1.0.0")
            .header("X-Device-ID", getDeviceId())
            .header("X-App-Version", BuildConfig.VERSION_NAME)
            .header("X-Platform", "Android")
            .build()
        
        return chain.proceed(newRequest)
    }
    
    private fun getDeviceId(): String {
        return Settings.Secure.getString(
            context.contentResolver,
            Settings.Secure.ANDROID_ID
        )
    }
}
```

### 🔧 Multiple Interceptors Setup:
```kotlin
@Provides
@Singleton
fun provideOkHttpClient(
    authInterceptor: AuthInterceptor,
    loggingInterceptor: HttpLoggingInterceptor
): OkHttpClient {
    return OkHttpClient.Builder()
        // Application interceptors (modify request/response)
        .addInterceptor(HeadersInterceptor())
        .addInterceptor(authInterceptor)
        .addInterceptor(CacheInterceptor())
        .addInterceptor(loggingInterceptor)
        
        // Network interceptors (see actual network calls)
        .addNetworkInterceptor(RetryInterceptor())
        
        // Timeouts
        .connectTimeout(30, TimeUnit.SECONDS)
        .readTimeout(30, TimeUnit.SECONDS)
        .writeTimeout(30, TimeUnit.SECONDS)
        
        .build()
}

// Execution Order:
// Request:  App → HeadersInterceptor → AuthInterceptor → CacheInterceptor 
//           → LoggingInterceptor → RetryInterceptor → Network
// Response: Network → RetryInterceptor → LoggingInterceptor 
//           → CacheInterceptor → AuthInterceptor → HeadersInterceptor → App
```

---

## Advanced Features

### 1️⃣ Response Caching
```kotlin
// Setup cache
val cacheSize = 10 * 1024 * 1024  // 10 MB
val cache = Cache(
    directory = File(context.cacheDir, "http_cache"),
    maxSize = cacheSize.toLong()
)

val client = OkHttpClient.Builder()
    .cache(cache)
    .build()

// Request with cache
val request = Request.Builder()
    .url("https://api.example.com/data")
    .cacheControl(
        CacheControl.Builder()
            .maxAge(5, TimeUnit.MINUTES)  // Cache for 5 minutes
            .build()
    )
    .build()

// First request → Server → Cache
// Second request (within 5 min) → Cache (no network!) ⚡
```

### 2️⃣ File Download
```kotlin
suspend fun downloadFile(
    url: String,
    outputFile: File
): Result<File> = withContext(Dispatchers.IO) {
    try {
        val request = Request.Builder()
            .url(url)
            .build()
        
        val response = client.newCall(request).execute()
        
        if (response.isSuccessful) {
            response.body?.byteStream()?.use { inputStream ->
                outputFile.outputStream().use { outputStream ->
                    inputStream.copyTo(outputStream)
                }
            }
            Result.success(outputFile)
        } else {
            Result.failure(Exception("Download failed: ${response.code}"))
        }
    } catch (e: Exception) {
        Result.failure(e)
    }
}

// Usage:
val file = File(context.filesDir, "downloaded.pdf")
downloadFile("https://example.com/file.pdf", file)
```

### 3️⃣ File Upload (Multipart)
```kotlin
suspend fun uploadImage(
    imageFile: File,
    description: String
): Result<String> = withContext(Dispatchers.IO) {
    try {
        // Create multipart body
        val requestBody = MultipartBody.Builder()
            .setType(MultipartBody.FORM)
            .addFormDataPart("description", description)
            .addFormDataPart(
                "image",
                imageFile.name,
                imageFile.asRequestBody("image/jpeg".toMediaType())
            )
            .build()
        
        // Build request
        val request = Request.Builder()
            .url("https://api.example.com/upload")
            .post(requestBody)
            .build()
        
        // Execute
        val response = client.newCall(request).execute()
        
        if (response.isSuccessful) {
            Result.success(response.body?.string() ?: "")
        } else {
            Result.failure(Exception("Upload failed: ${response.code}"))
        }
    } catch (e: Exception) {
        Result.failure(e)
    }
}
```

### 4️⃣ WebSocket Connection
```kotlin
class WebSocketManager(
    private val client: OkHttpClient
) {
    
    private var webSocket: WebSocket? = null
    
    fun connect(url: String, listener: WebSocketListener) {
        val request = Request.Builder()
            .url(url)
            .build()
        
        webSocket = client.newWebSocket(request, listener)
    }
    
    fun send(message: String) {
        webSocket?.send(message)
    }
    
    fun disconnect() {
        webSocket?.close(1000, "Goodbye")
    }
}

// Usage:
val listener = object : WebSocketListener() {
    override fun onOpen(webSocket: WebSocket, response: Response) {
        println("Connected!")
    }
    
    override fun onMessage(webSocket: WebSocket, text: String) {
        println("Received: $text")
    }
    
    override fun onClosed(webSocket: WebSocket, code: Int, reason: String) {
        println("Closed: $reason")
    }
    
    override fun onFailure(webSocket: WebSocket, t: Throwable, response: Response?) {
        println("Error: ${t.message}")
    }
}

val wsManager = WebSocketManager(client)
wsManager.connect("wss://echo.websocket.org", listener)
wsManager.send("Hello WebSocket!")
```

### 5️⃣ Connection Pool Configuration
```kotlin
val client = OkHttpClient.Builder()
    .connectionPool(
        ConnectionPool(
            maxIdleConnections = 5,      // Max idle connections
            keepAliveDuration = 5,        // Keep alive duration
            timeUnit = TimeUnit.MINUTES   // Time unit
        )
    )
    .build()

// Benefits:
// ✅ Reuses connections
// ✅ Faster subsequent requests
// ✅ Less battery usage
```

### 6️⃣ DNS Configuration
```kotlin
val client = OkHttpClient.Builder()
    .dns(object : Dns {
        override fun lookup(hostname: String): List<InetAddress> {
            // Custom DNS logic
            return Dns.SYSTEM.lookup(hostname)
        }
    })
    .build()
```

---

## Real World Examples

### 🏗️ Complete Repository Example
```kotlin
// data/repository/UserRepository.kt

interface UserRepository {
    suspend fun getUsers(): Result<List<User>>
    suspend fun getUserById(id: String): Result<User>
    suspend fun createUser(user: User): Result<User>
    suspend fun updateUser(id: String, user: User): Result<User>
    suspend fun deleteUser(id: String): Result<Unit>
}

class UserRepositoryImpl @Inject constructor(
    private val client: OkHttpClient,
    private val gson: Gson
) : UserRepository {
    
    private val baseUrl = "https://jsonplaceholder.typicode.com"
    
    override suspend fun getUsers(): Result<List<User>> {
        return withContext(Dispatchers.IO) {
            try {
                val request = Request.Builder()
                    .url("$baseUrl/users")
                    .get()
                    .build()
                
                val response = client.newCall(request).execute()
                
                if (response.isSuccessful) {
                    val json = response.body?.string() ?: "[]"
                    val users = gson.fromJson(json, Array<User>::class.java).toList()
                    Result.success(users)
                } else {
                    Result.failure(Exception("Error: ${response.code}"))
                }
            } catch (e: Exception) {
                Result.failure(e)
            }
        }
    }
    
    override suspend fun getUserById(id: String): Result<User> {
        return withContext(Dispatchers.IO) {
            try {
                val request = Request.Builder()
                    .url("$baseUrl/users/$id")
                    .get()
                    .build()
                
                val response = client.newCall(request).execute()
                
                if (response.isSuccessful) {
                    val json = response.body?.string() ?: "{}"
                    val user = gson.fromJson(json, User::class.java)
                    Result.success(user)
                } else {
                    Result.failure(Exception("User not found"))
                }
            } catch (e: Exception) {
                Result.failure(e)
            }
        }
    }
    
    override suspend fun createUser(user: User): Result<User> {
        return withContext(Dispatchers.IO) {
            try {
                val json = gson.toJson(user)
                val mediaType = "application/json".toMediaType()
                val body = json.toRequestBody(mediaType)
                
                val request = Request.Builder()
                    .url("$baseUrl/users")
                    .post(body)
                    .build()
                
                val response = client.newCall(request).execute()
                
                if (response.isSuccessful) {
                    val responseJson = response.body?.string() ?: "{}"
                    val createdUser = gson.fromJson(responseJson, User::class.java)
                    Result.success(createdUser)
                } else {
                    Result.failure(Exception("Failed to create user"))
                }
            } catch (e: Exception) {
                Result.failure(e)
            }
        }
    }
    
    override suspend fun updateUser(id: String, user: User): Result<User> {
        return withContext(Dispatchers.IO) {
            try {
                val json = gson.toJson(user)
                val mediaType = "application/json".toMediaType()
                val body = json.toRequestBody(mediaType)
                
                val request = Request.Builder()
                    .url("$baseUrl/users/$id")
                    .put(body)
                    .build()
                
                val response = client.newCall(request).execute()
                
                if (response.isSuccessful) {
                    val responseJson = response.body?.string() ?: "{}"
                    val updatedUser = gson.fromJson(responseJson, User::class.java)
                    Result.success(updatedUser)
                } else {
                    Result.failure(Exception("Failed to update user"))
                }
            } catch (e: Exception) {
                Result.failure(e)
            }
        }
    }
    
    override suspend fun deleteUser(id: String): Result<Unit> {
        return withContext(Dispatchers.IO) {
            try {
                val request = Request.Builder()
                    .url("$baseUrl/users/$id")
                    .delete()
                    .build()
                
                val response = client.newCall(request).execute()
                
                if (response.isSuccessful) {
                    Result.success(Unit)
                } else {
                    Result.failure(Exception("Failed to delete user"))
                }
            } catch (e: Exception) {
                Result.failure(e)
            }
        }
    }
}
```

### 🎯 Complete Setup with Hilt
```kotlin
// data/di/NetworkModule.kt

@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {
    
    @Provides
    @Singleton
    fun provideGson(): Gson {
        return GsonBuilder()
            .setLenient()
            .create()
    }
    
    @Provides
    @Singleton
    fun provideAuthInterceptor(
        tokenManager: TokenManager
    ): AuthInterceptor {
        return AuthInterceptor(tokenManager)
    }
    
    @Provides
    @Singleton
    fun provideLoggingInterceptor(): HttpLoggingInterceptor {
        return HttpLoggingInterceptor().apply {
            level = if (BuildConfig.DEBUG) {
                HttpLoggingInterceptor.Level.BODY
            } else {
                HttpLoggingInterceptor.Level.NONE
            }
        }
    }
    
    @Provides
    @Singleton
    fun provideCache(
        @ApplicationContext context: Context
    ): Cache {
        val cacheSize = 10 * 1024 * 1024L  // 10 MB
        return Cache(
            directory = File(context.cacheDir, "http_cache"),
            maxSize = cacheSize
        )
    }
    
    @Provides
    @Singleton
    fun provideOkHttpClient(
        authInterceptor: AuthInterceptor,
        loggingInterceptor: HttpLoggingInterceptor,
        cache: Cache
    ): OkHttpClient {
        return OkHttpClient.Builder()
            // Cache
            .cache(cache)
            
            // Interceptors
            .addInterceptor(HeadersInterceptor())
            .addInterceptor(authInterceptor)
            .addInterceptor(ErrorHandlingInterceptor())
            .addInterceptor(loggingInterceptor)
            
            // Timeouts
            .connectTimeout(30, TimeUnit.SECONDS)
            .readTimeout(30, TimeUnit.SECONDS)
            .writeTimeout(30, TimeUnit.SECONDS)
            
            // Connection pool
            .connectionPool(
                ConnectionPool(
                    maxIdleConnections = 5,
                    keepAliveDuration = 5,
                    timeUnit = TimeUnit.MINUTES
                )
            )
            
            // Retry
            .retryOnConnectionFailure(true)
            
            .build()
    }
}

// data/di/RepositoryModule.kt

@Module
@InstallIn(SingletonComponent::class)
abstract class RepositoryModule {
    
    @Binds
    @Singleton
    abstract fun bindUserRepository(
        impl: UserRepositoryImpl
    ): UserRepository
}
```

---

## Best Practices

### ✅ 1. Use Singleton OkHttpClient
```kotlin
// ✅ GOOD - Single instance
@Singleton
@Provides
fun provideOkHttpClient(): OkHttpClient {
    return OkHttpClient.Builder()
        .build()
}

// ❌ BAD - New instance every time
fun makeRequest() {
    val client = OkHttpClient()  // Don't do this!
    // ...
}

// Why?
// ✅ Singleton: Connection pooling works
// ❌ Multiple: No connection reuse, slow, memory waste
```

### ✅ 2. Always Use Coroutines
```kotlin
// ✅ GOOD - Coroutines
suspend fun getUsers(): Result<List<User>> {
    return withContext(Dispatchers.IO) {
        // Network call
    }
}

// ❌ BAD - Main thread
fun getUsers() {
    val response = client.newCall(request).execute()  // NetworkOnMainThreadException!
}
```

### ✅ 3. Handle Errors Properly
```kotlin
// ✅ GOOD - Comprehensive error handling
try {
    val response = client.newCall(request).execute()
    
    when {
        response.isSuccessful -> {
            // Handle success
        }
        response.code == 401 -> {
            // Handle unauthorized
        }
        response.code in 500..599 -> {
            // Handle server error
        }
        else -> {
            // Handle other errors
        }
    }
} catch (e: IOException) {
    // Handle network error
} catch (e: Exception) {
    // Handle unexpected error
}
```

### ✅ 4. Close Response Bodies
```kotlin
// ✅ GOOD - Auto close with use
client.newCall(request).execute().use { response ->
    val body = response.body?.string()
    // Response automatically closed
}

// ❌ BAD - Might not close
val response = client.newCall(request).execute()
val body = response.body?.string()
// Response body might leak!
```

### ✅ 5. Set Appropriate Timeouts
```kotlin
// ✅ GOOD - Reasonable timeouts
val client = OkHttpClient.Builder()
    .connectTimeout(30, TimeUnit.SECONDS)  // Connection
    .readTimeout(30, TimeUnit.SECONDS)     // Reading data
    .writeTimeout(30, TimeUnit.SECONDS)    // Writing data
    .build()

// ❌ BAD - No timeout (might hang forever!)
val client = OkHttpClient()
```

### ✅ 6. Use Logging Interceptor Wisely
```kotlin
// ✅ GOOD - Debug only
val loggingInterceptor = HttpLoggingInterceptor().apply {
    level = if (BuildConfig.DEBUG) {
        HttpLoggingInterceptor.Level.BODY
    } else {
        HttpLoggingInterceptor.Level.NONE  // No logs in production
    }
}

// Why?
// ✅ Debug: See all requests/responses
// ✅ Production: No performance overhead, no sensitive data leaks
```

### ✅ 7. Implement Retry Logic
```kotlin
// ✅ GOOD - Auto retry
val client = OkHttpClient.Builder()
    .retryOnConnectionFailure(true)  // Retry on connection failure
    .addInterceptor(RetryInterceptor(maxRetries = 3))
    .build()
```

### ✅ 8. Use Connection Pooling
```kotlin
// ✅ GOOD - Reuse connections
val client = OkHttpClient.Builder()
    .connectionPool(
        ConnectionPool(
            maxIdleConnections = 5,
            keepAliveDuration = 5,
            timeUnit = TimeUnit.MINUTES
        )
    )
    .build()
```

---

## Troubleshooting

### ❌ Problem 1: NetworkOnMainThreadException
```kotlin
Error: android.os.NetworkOnMainThreadException

✅ Solution: Use Coroutines
suspend fun getData() = withContext(Dispatchers.IO) {
    client.newCall(request).execute()
}
```

### ❌ Problem 2: SocketTimeoutException
```kotlin
Error: java.net.SocketTimeoutException: timeout

✅ Solution: Increase timeout
val client = OkHttpClient.Builder()
    .connectTimeout(60, TimeUnit.SECONDS)
    .readTimeout(60, TimeUnit.SECONDS)
    .build()
```

### ❌ Problem 3: UnknownHostException
```kotlin
Error: java.net.UnknownHostException

✅ Causes:
1. No internet connection
2. Wrong URL
3. DNS issue

✅ Solution:
- Check internet permission
- Verify URL is correct
- Handle error gracefully
```

### ❌ Problem 4: SSLHandshakeException
```kotlin
Error: javax.net.ssl.SSLHandshakeException

✅ Solution:
// For development only! Never in production!
val client = OkHttpClient.Builder()
    .hostnameVerifier { _, _ -> true }  // Bypass SSL (dev only!)
    .build()

// Better: Fix server's SSL certificate
```

### ❌ Problem 5: Too Many Requests (429)
```kotlin
Error: HTTP 429 Too Many Requests

✅ Solution: Implement rate limiting
class RateLimitInterceptor : Interceptor {
    private var lastRequestTime = 0L
    private val minInterval = 1000L  // 1 second
    
    override fun intercept(chain: Interceptor.Chain): Response {
        val now = System.currentTimeMillis()
        val timeSinceLastRequest = now - lastRequestTime
        
        if (timeSinceLastRequest < minInterval) {
            Thread.sleep(minInterval - timeSinceLastRequest)
        }
        
        lastRequestTime = System.currentTimeMillis()
        return chain.proceed(chain.request())
    }
}
```

---

## 📝 Quick Reference

### Essential Code Snippets:
```kotlin
// 1. Simple GET
val request = Request.Builder()
    .url("https://api.example.com/data")
    .build()

// 2. POST with JSON
val json = JSONObject().apply { put("key", "value") }
val body = json.toString().toRequestBody("application/json".toMediaType())
val request = Request.Builder()
    .url("https://api.example.com/data")
    .post(body)
    .build()

// 3. With Headers
val request = Request.Builder()
    .url("https://api.example.com/data")
    .header("Authorization", "Bearer token")
    .build()

// 4. Execute
val response = client.newCall(request).execute()
if (response.isSuccessful) {
    val data = response.body?.string()
}
```

### Common Response Codes:

| Code | Meaning | Action |
|------|---------|--------|
| 200 | OK | Success! |
| 201 | Created | Resource created |
| 204 | No Content | Success, no data |
| 400 | Bad Request | Check request data |
| 401 | Unauthorized | Login required |
| 403 | Forbidden | No permission |
| 404 | Not Found | Resource doesn't exist |
| 429 | Too Many Requests | Rate limited |
| 500 | Server Error | Server problem |
| 503 | Service Unavailable | Server down |

---

## 🎓 Key Takeaways
```
✅ OkHttp = Powerful HTTP client
✅ Better than HttpURLConnection
✅ Automatic connection pooling
✅ Built-in caching
✅ Interceptors for customization
✅ Thread-safe
✅ HTTP/2 support

Remember:
📌 Use singleton OkHttpClient
📌 Always use Dispatchers.IO
📌 Handle errors properly
📌 Close response bodies
📌 Use logging in debug only
📌 Set appropriate timeouts
📌 Implement retry logic

Common Pattern:
Request → OkHttpClient → Interceptors → Network → Response
```

---

## 🔗 Resources

- [Official Documentation](https://square.github.io/okhttp/)
- [GitHub Repository](https://github.com/square/okhttp)
- [Recipes/Examples](https://square.github.io/okhttp/recipes/)

---

## 💡 Why OkHttp with Retrofit?
```
Many developers ask: "Should I use OkHttp or Retrofit?"

Answer: Use BOTH! 🎯

Retrofit = High-level API interface
   ↓ (uses internally)
OkHttp = Low-level HTTP client

Example:
interface ApiService {
    @GET("users")
    suspend fun getUsers(): List<User>
}

Retrofit Builder:
    .client(okHttpClient)  ← Your custom OkHttpClient!
    .build()

Benefits:
✅ Retrofit handles API interface
✅ OkHttp handles connection, caching, interceptors
✅ Best of both worlds!
```

---

*Happy Coding with OkHttp! 🚀*

---

*Last Updated: December 2024*
