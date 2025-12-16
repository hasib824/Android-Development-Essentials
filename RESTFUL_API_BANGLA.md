# RESTful API - সহজ বাংলা টিউটোরিয়াল

## 📚 সূচিপত্র
1. [REST API কি?](#rest-api-কি)
2. [মূল নীতিসমূহ](#মূল-নীতিসমূহ)
3. [HTTP Methods](#http-methods)
4. [Status Codes](#status-codes)
5. [URL Design](#url-design)
6. [Real Examples](#real-examples)
7. [Best Practices](#best-practices)

---

## REST API কি?

**REST** = **RE**presentational **S**tate **T**ransfer

### 🎯 এক কথায়:
```
REST API = Apps যেভাবে একে অপরের সাথে কথা বলে

আপনার Mobile App → REST API → Server
                        ↓
                   JSON Data আসে
```

### 💡 Real Life উদাহরণ:
```
Restaurant এ Order দেওয়া:

1. আপনি Waiter কে বলেন: "Pizza চাই" (Request)
2. Waiter Kitchen এ যায় (API)
3. Chef Pizza বানায় (Server)
4. Waiter Pizza নিয়ে আসে (Response)

REST API ঠিক এভাবে কাজ করে!

Mobile App: "Users এর list চাই"
     ↓
REST API: Server থেকে নিয়ে আসে
     ↓
Mobile App: List দেখায়
```

### 🔑 মূল Concept:
```
REST = Resources নিয়ে কাজ করে

Resource = যেকোনো Data/Object
Examples: User, Post, Product, Order

প্রতিটা Resource এর একটা unique URL থাকে:
/users      → সব users
/users/123  → User 123
/posts      → সব posts
/posts/456  → Post 456
```

---

## মূল নীতিসমূহ

### 1️⃣ Client-Server (আলাদা থাকে)
```
┌─────────────┐         ┌─────────────┐
│   Mobile    │ ◄─────► │   Server    │
│    App      │  HTTP   │  Database   │
└─────────────┘         └─────────────┘

Mobile App: শুধু UI দেখায়
Server: শুধু Data রাখে

একে অপরের কাজে হস্তক্ষেপ করে না ✅
```

### 2️⃣ Stateless (Memory রাখে না)
```
Request 1: "User 123 চাই"
Server: ✅ দিলাম
Server: 🧹 ভুলে গেলাম

Request 2: "Posts চাই"
Server: ✅ দিলাম
Server: 🧹 আবার ভুলে গেলাম

Server কখনো remember করে না আগের request!

Benefits:
✅ Simple
✅ Scalable
✅ Fast
```

### 3️⃣ Cacheable (Cache করা যায়)
```
First Time:
Request → Server → Response → Cache এ save ✅

Next Time (same request):
Request → Cache থেকে দেয় ⚡ (অনেক Fast!)

No need to hit server again!
```

### 4️⃣ Uniform Interface (একই Pattern)
```
সব Resource এর জন্য same rules:

Users:
GET    /users      → সব users
GET    /users/123  → একটা user
POST   /users      → নতুন user
DELETE /users/123  → user মুছো

Posts:
GET    /posts      → সব posts
GET    /posts/456  → একটা post
POST   /posts      → নতুন post
DELETE /posts/456  → post মুছো

Pattern same! শুধু resource name change 🎯
```

---

## HTTP Methods

HTTP Methods বলে **কি করতে চান**।

### 📊 Visual Summary:
```
┌─────────────────────────────────────────────┐
│  Method   │   কাজ    │   Example           │
├─────────────────────────────────────────────┤
│  GET      │  পড়ো    │  List দেখাও         │
│  POST     │  বানাও   │  নতুন তৈরি করো      │
│  PUT      │  Replace │  পুরোটা বদলাও       │
│  PATCH    │  Update  │  কিছু বদলাও         │
│  DELETE   │  মুছো    │  Delete করো         │
└─────────────────────────────────────────────┘
```

### 1️⃣ GET - পড়া (Read)
```kotlin
// সব users পাও
GET /api/users

Response:
[
  { "id": 1, "name": "Hasib" },
  { "id": 2, "name": "Rahim" }
]

// একটা user পাও
GET /api/users/123

Response:
{
  "id": 123,
  "name": "Hasib",
  "email": "hasib@example.com"
}

Characteristics:
✅ Safe (কিছু change করে না)
✅ Cacheable
❌ Body নেই
```

### 2️⃣ POST - তৈরি করা (Create)
```kotlin
// নতুন user বানাও
POST /api/users

Body:
{
  "name": "Hasib",
  "email": "hasib@example.com"
}

Response (201 Created):
{
  "id": 456,  // ← নতুন ID
  "name": "Hasib",
  "email": "hasib@example.com"
}

Characteristics:
❌ Not safe (নতুন data তৈরি করে)
✅ Body আছে
✅ Response এ নতুন ID থাকে
```

### 3️⃣ PUT - সব Replace করা (Replace All)
```kotlin
// User এর সব info বদলাও
PUT /api/users/123

Body (সব fields দিতে হবে):
{
  "name": "Hasibuzzaman",
  "email": "new@example.com",
  "age": 26,
  "city": "Dhaka"
}

Response (200 OK):
{
  "id": 123,
  "name": "Hasibuzzaman",  // ← Updated
  "email": "new@example.com",  // ← Updated
  "age": 26,  // ← Updated
  "city": "Dhaka"  // ← Updated
}

⚠️ যা দেননি সেটা null হয়ে যাবে!
```

### 4️⃣ PATCH - কিছু Update করা (Partial Update)
```kotlin
// শুধু name বদলাও
PATCH /api/users/123

Body (শুধু যা বদলাতে চান):
{
  "name": "Hasibuzzaman"
}

Response (200 OK):
{
  "id": 123,
  "name": "Hasibuzzaman",  // ← Updated ✅
  "email": "old@example.com",  // ← Unchanged ✅
  "age": 25,  // ← Unchanged ✅
  "city": "Dhaka"  // ← Unchanged ✅
}

✅ বাকি সব same থাকবে!
```

### 5️⃣ DELETE - মুছে ফেলা (Delete)
```kotlin
// User মুছে ফেলো
DELETE /api/users/123

Response (204 No Content):
(Empty - no body)

অথবা

Response (200 OK):
{
  "message": "User deleted successfully"
}
```

### 🔥 PUT vs PATCH - পার্থক্য:
```
Original User:
{
  "name": "Hasib",
  "email": "hasib@example.com",
  "age": 25
}

───────────────────────────────────────

PUT (সব replace):
{
  "name": "New Name"
  // email এবং age missing!
}

Result:
{
  "name": "New Name",
  "email": null,  // ← Lost! 😱
  "age": null  // ← Lost! 😱
}

───────────────────────────────────────

PATCH (কিছু update):
{
  "name": "New Name"
}

Result:
{
  "name": "New Name",  // ← Updated ✅
  "email": "hasib@example.com",  // ← Safe ✅
  "age": 25  // ← Safe ✅
}
```

---

## Status Codes

Status Codes বলে **কি হলো**।

### 📊 Categories:
```
2xx = ✅ Success (সফল হয়েছে)
4xx = ❌ Your Mistake (আপনার ভুল)
5xx = 🔥 Server Error (সার্ভারের সমস্যা)
```

### ✅ Success Codes (2xx):
```
200 OK
  সব ঠিক আছে!
  
  Example:
  GET /api/users/123 → 200 OK
  {
    "id": 123,
    "name": "Hasib"
  }

201 Created
  নতুন তৈরি হয়েছে!
  
  Example:
  POST /api/users → 201 Created
  {
    "id": 456,
    "name": "New User"
  }

204 No Content
  সফল কিন্তু কিছু return করার নেই
  
  Example:
  DELETE /api/users/123 → 204 No Content
  (Empty body)
```

### ❌ Client Error Codes (4xx):
```
400 Bad Request
  আপনার request ভুল
  
  Example: Invalid data
  POST /api/users
  Body: { "name": "" }  // ← Empty!
  → 400 Bad Request

401 Unauthorized
  Login করেননি
  
  Example: No token
  GET /api/profile
  → 401 Unauthorized

403 Forbidden
  Permission নেই
  
  Example: Not admin
  DELETE /api/users/123
  → 403 Forbidden

404 Not Found
  খুঁজে পাওয়া যায়নি
  
  Example: Wrong ID
  GET /api/users/99999
  → 404 Not Found

409 Conflict
  Already আছে (duplicate)
  
  Example: Email exists
  POST /api/users
  Body: { "email": "existing@example.com" }
  → 409 Conflict
```

### 🔥 Server Error Codes (5xx):
```
500 Internal Server Error
  Server এ সমস্যা
  
  Example: Database crash
  GET /api/users
  → 500 Internal Server Error

503 Service Unavailable
  Server চলছে না
  
  Example: Maintenance
  GET /api/users
  → 503 Service Unavailable
```

### 📌 Quick Reference:

| Code | Meaning | কখন |
|------|---------|-----|
| **200** | OK | GET, PUT, PATCH সফল |
| **201** | Created | POST সফল |
| **204** | No Content | DELETE সফল |
| **400** | Bad Request | Invalid data |
| **401** | Unauthorized | Login লাগবে |
| **404** | Not Found | খুঁজে পাওয়া যায়নি |
| **500** | Server Error | Server সমস্যা |

---

## URL Design

### ✅ Good URL Structure:
```
Pattern:
/api/{version}/{resource}/{id}

Examples:
✅ /api/v1/users
✅ /api/v1/users/123
✅ /api/v1/users/123/posts
✅ /api/v1/posts?author_id=123
```

### 🎯 Rules - মনে রাখুন:

#### 1. Nouns ব্যবহার করুন (Verbs না)
```
✅ GOOD - Resource names
/api/users
/api/posts
/api/products

❌ BAD - Action names
/api/getUsers
/api/createPost
/api/deleteProduct

কেন? HTTP methods already বলে দেয় action!
```

#### 2. Plural ব্যবহার করুন
```
✅ GOOD
/api/users
/api/posts

❌ BAD
/api/user
/api/post
```

#### 3. Nested Resources
```
✅ GOOD
/api/users/123/posts        (User 123 এর posts)
/api/posts/456/comments     (Post 456 এর comments)

Pattern:
/parent-resource/id/child-resource
```

#### 4. Query Parameters - Filtering/Sorting
```
Filtering:
/api/users?age=25
/api/users?city=Dhaka&status=active

Sorting:
/api/users?sort=name&order=asc
/api/posts?sort=created_at&order=desc

Pagination:
/api/users?page=1&limit=20

Search:
/api/users?search=hasib
```

#### 5. Lowercase এবং Hyphens
```
✅ GOOD
/api/user-profiles
/api/blog-posts

❌ BAD
/api/UserProfiles
/api/blog_posts
```

### 📋 Complete Example:
```
Users Resource:

GET    /api/v1/users                  → All users
GET    /api/v1/users?page=2           → Page 2
GET    /api/v1/users?search=hasib     → Search
GET    /api/v1/users/123              → User 123
POST   /api/v1/users                  → Create
PUT    /api/v1/users/123              → Full update
PATCH  /api/v1/users/123              → Partial update
DELETE /api/v1/users/123              → Delete

User's Posts:

GET    /api/v1/users/123/posts        → User's posts
POST   /api/v1/users/123/posts        → Create post
GET    /api/v1/users/123/posts/456    → Specific post
DELETE /api/v1/users/123/posts/456    → Delete post
```

---

## Real Examples

### 📱 Example 1: Social Media App
```kotlin
// Users
GET    /api/v1/users              → সব users
GET    /api/v1/users/123          → User profile
PATCH  /api/v1/users/123          → Profile update

// Posts
GET    /api/v1/posts              → Feed (সব posts)
GET    /api/v1/posts?user_id=123  → User's posts
POST   /api/v1/posts              → নতুন post
DELETE /api/v1/posts/456          → Post delete

// Comments
GET    /api/v1/posts/456/comments     → Post এর comments
POST   /api/v1/posts/456/comments     → নতুন comment
DELETE /api/v1/posts/456/comments/789 → Comment delete

// Likes
POST   /api/v1/posts/456/like     → Post like করো
DELETE /api/v1/posts/456/like     → Unlike করো
```

### 🛒 Example 2: E-commerce App
```kotlin
// Products
GET    /api/v1/products                          → সব products
GET    /api/v1/products?category=electronics     → Filter
GET    /api/v1/products/123                      → Product details

// Cart
GET    /api/v1/cart                  → My cart
POST   /api/v1/cart/items            → Add to cart
PATCH  /api/v1/cart/items/456        → Update quantity
DELETE /api/v1/cart/items/456        → Remove from cart

// Orders
GET    /api/v1/orders                → My orders
GET    /api/v1/orders/789            → Order details
POST   /api/v1/orders                → Place order
```

---

## Best Practices

### 1️⃣ Consistent নামকরণ
```
✅ GOOD
/api/users
/api/posts
/api/comments

❌ BAD
/api/users
/api/getPost
/api/comment-list
```

### 2️⃣ Versioning ব্যবহার করুন
```
✅ GOOD
/api/v1/users
/api/v2/users

কেন?
Breaking changes হলে v2 বানান
v1 users still কাজ করবে! ✅
```

### 3️⃣ Pagination করুন
```
✅ GOOD
GET /api/users?page=1&limit=20

Response:
{
  "data": [...],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 150
  }
}

❌ BAD
GET /api/users
(10,000 users একসাথে! 😱)
```

### 4️⃣ Error Messages Clear রাখুন
```
✅ GOOD
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Email is required",
    "field": "email"
  }
}

❌ BAD
{
  "error": "Error occurred"
}
```

### 5️⃣ HTTPS ব্যবহার করুন
```
❌ http://api.example.com
✅ https://api.example.com

Security! 🔒
```

### 6️⃣ Authentication
```
Most Common: Bearer Token

Login:
POST /api/auth/login
Body: { "email": "...", "password": "..." }

Response:
{
  "token": "eyJhbGc..."
}

Subsequent Requests:
GET /api/users
Headers:
  Authorization: Bearer eyJhbGc...
```

---

## 📝 Quick Cheat Sheet

### HTTP Methods:
```
GET    → পড়ো (Read)
POST   → বানাও (Create)
PUT    → Replace করো (Replace All)
PATCH  → Update করো (Update Some)
DELETE → মুছো (Delete)
```

### Status Codes:
```
2xx → ✅ Success
  200 OK
  201 Created
  204 No Content

4xx → ❌ Your Error
  400 Bad Request
  401 Unauthorized
  404 Not Found

5xx → 🔥 Server Error
  500 Internal Server Error
```

### URL Pattern:
```
/api/v1/resource
/api/v1/resource/id
/api/v1/resource/id/sub-resource
```

---

## 🎓 মূল শিক্ষা
```
╔════════════════════════════════════════════╗
║         REST API - Core Concepts          ║
╚════════════════════════════════════════════╝

1. REST = Resources নিয়ে কাজ
   Resource = User, Post, Product, etc.

2. HTTP Methods = কি করতে চাই
   GET, POST, PUT, PATCH, DELETE

3. Status Codes = কি হলো
   2xx = Success, 4xx = Your error, 5xx = Server error

4. URL = Resource এর address
   /api/v1/users/123

5. Stateless = Server কিছু মনে রাখে না

6. JSON = Data format
   Request/Response both JSON

Golden Rules:
✅ Simple রাখুন
✅ Consistent থাকুন
✅ HTTP standards follow করুন
✅ Version করুন
✅ Document করুন
```

---

## 🔄 Request-Response Flow
```
┌─────────────────────────────────────────────┐
│         Complete Flow Example               │
└─────────────────────────────────────────────┘

Mobile App:
  "User 123 চাই"
       ↓
  
Request:
  GET /api/v1/users/123
  Headers:
    Authorization: Bearer token...
       ↓

REST API (Server):
  1. Token check করে ✅
  2. Database query করে
  3. Data পায়
       ↓

Response:
  Status: 200 OK
  Body:
  {
    "id": 123,
    "name": "Hasib",
    "email": "hasib@example.com"
  }
       ↓

Mobile App:
  User profile দেখায় ✅
```

---

## 💡 Remember This!
```
REST API = Restaurant Menu

Menu তে থাকে:
- কি কি আছে (Resources)
- কিভাবে order করবেন (HTTP Methods)
- কত সময় লাগবে (Response Time)

REST API তে থাকে:
- কি কি data আছে (Users, Posts, etc)
- কিভাবে request করবেন (GET, POST, etc)
- কি response আসবে (JSON data)

Simple! 🎯
```

---

*Happy API Development! 🚀*

*Last Updated: December 2024*
