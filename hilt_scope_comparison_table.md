# 🎯 Hilt Scopes - Complete Comparison Table

## 📊 সব Scope একসাথে - Complete Overview

| Scope Name | @InstallIn Signature | Annotation | Lifetime | Screen Rotation | Memory Impact | Primary Use Cases |
|------------|---------------------|------------|----------|-----------------|---------------|-------------------|
| **Singleton** | `SingletonComponent::class` | `@Singleton` | পুরো Application lifetime (app open to close) | ✅ Survives | **Low** (1 instance for entire app) | • Database<br>• Retrofit/OkHttp<br>• SharedPreferences<br>• DataStore<br>• Global Config<br>• Analytics SDK |
| **ActivityRetained** | `ActivityRetainedComponent::class` | `@ActivityRetainedScoped` | Activity lifetime + Configuration changes survive | ✅ Survives | **Medium** (1 instance per activity, survives rotation) | • Shopping Cart<br>• Checkout Session<br>• Multi-step Form Data<br>• Wizard Progress<br>• Temp Upload Queue<br>• Filter State |
| **ViewModel** | `ViewModelComponent::class` | `@ViewModelScoped` | ViewModel lifetime (until cleared) | ✅ Survives | **Low** (1 instance per ViewModel) | • Screen Analytics<br>• Form Validators<br>• Screen Cache<br>• Data Processors<br>• Screen State Managers |
| **Activity** | `ActivityComponent::class` | `@ActivityScoped` | Activity onCreate to onDestroy | ❌ Recreated | **Medium** (1 instance per activity lifecycle) | • Cart Badge Manager<br>• Navigation Helper<br>• Snackbar Manager<br>• Dialog Coordinator<br>• Permission Handler<br>• Bottom Sheet Manager |
| **Fragment** | `FragmentComponent::class` | `@FragmentScoped` | Fragment onCreate to onDestroy | ❌ Recreated | **Low** (1 instance per fragment, cleared when not visible) | • RecyclerView Adapter<br>• Fragment Dialogs<br>• Image Loaders<br>• Fragment Animations<br>• ViewPager Helpers |
| **Service** | `ServiceComponent::class` | `@ServiceScoped` | Service onCreate to onDestroy | N/A (Services don't rotate) | **Medium** (1 instance per service) | • Music Player<br>• Download Manager<br>• Location Tracker<br>• Sync Services<br>• Background Tasks |

---

## 🔄 Rotation Behavior - বিস্তারিত

| Scope | Screen Rotate করলে কি হয়? | Data থাকবে? | Instance কি হবে? |
|-------|---------------------------|-------------|------------------|
| **@Singleton** | কিছু হয় না | ✅ YES - সব data intact | Same instance |
| **@ActivityRetainedScoped** | কিছু হয় না | ✅ YES - সব data intact | Same instance |
| **@ViewModelScoped** | কিছু হয় না | ✅ YES - সব data intact | Same instance (ViewModel survives) |
| **@ActivityScoped** | Destroy + Recreate হয় | ❌ NO - data lost | New instance তৈরি হবে |
| **@FragmentScoped** | Destroy + Recreate হয় | ❌ NO - data lost | New instance তৈরি হবে |
| **@ServiceScoped** | N/A | N/A | Services rotate করে না |

---

## 💾 Memory Impact - সংখ্যা সহ

| Scope | App এ কতগুলো Instance? | Memory Footprint | কখন Clear হয়? |
|-------|----------------------|------------------|---------------|
| **@Singleton** | 1 (পুরো app এ একটাই) | **Smallest** - শুধু 1 instance | App close/kill হলে |
| **@ActivityRetainedScoped** | N (প্রতি activity তে 1) | **Medium** - সাধারণত 1-3 instances (1-3 activities open থাকলে) | Activity finish হলে |
| **@ViewModelScoped** | N (প্রতি ViewModel তে 1) | **Small** - সাধারণত 3-10 instances | ViewModel clear হলে |
| **@ActivityScoped** | N (প্রতি activity তে 1) | **Medium** - active activities অনুযায়ী | Activity destroy হলে |
| **@FragmentScoped** | N (প্রতি fragment তে 1) | **Smallest** - ViewPager clear করে দেয় | Fragment destroy হলে |
| **@ServiceScoped** | N (প্রতি service তে 1) | **Medium** - সাধারণত 1-2 instances | Service stop হলে |

### 📊 Example Calculation - E-commerce App:

```
একটা e-commerce app এ Database inject করা আছে 10 জায়গায়:

❌ Without @Singleton:
10 screens × Database instance = 10 databases × 50MB = 500MB ⚠️

✅ With @Singleton:
1 Database instance = 50MB ✅ (450MB saved! 90% reduction)
```

---

## 🎯 Module Declaration - প্রতিটার জন্য Template

### 1. Singleton Module
```kotlin
@Module
@InstallIn(SingletonComponent::class)
object DatabaseModule {
    
    @Provides
    @Singleton
    fun provideDatabase(@ApplicationContext context: Context): AppDatabase {
        return Room.databaseBuilder(context, AppDatabase::class.java, "db").build()
    }
}
```

### 2. ActivityRetainedScoped Module
```kotlin
@Module
@InstallIn(ActivityRetainedComponent::class)
object SessionModule {
    
    @Provides
    @ActivityRetainedScoped
    fun provideCheckoutSession(): CheckoutSession {
        return CheckoutSession()
    }
}
```

### 3. ViewModelScoped Module
```kotlin
@Module
@InstallIn(ViewModelComponent::class)
object AnalyticsModule {
    
    @Provides
    @ViewModelScoped
    fun provideScreenAnalytics(): ScreenAnalytics {
        return ScreenAnalytics()
    }
}
```

### 4. ActivityScoped Module
```kotlin
@Module
@InstallIn(ActivityComponent::class)
object NavigationModule {
    
    @Provides
    @ActivityScoped
    fun provideNavigationHelper(activity: Activity): NavigationHelper {
        return NavigationHelper(activity)
    }
}
```

### 5. FragmentScoped Module
```kotlin
@Module
@InstallIn(FragmentComponent::class)
object AdapterModule {
    
    @Provides
    @FragmentScoped
    fun provideProductAdapter(): ProductAdapter {
        return ProductAdapter()
    }
}
```

### 6. ServiceScoped Module
```kotlin
@Module
@InstallIn(ServiceComponent::class)
object PlayerModule {
    
    @Provides
    @ServiceScoped
    fun provideMusicPlayer(service: Service): MusicPlayer {
        return MusicPlayer(service)
    }
}
```

---

## ⚡ Performance Comparison

| Scope | Creation Time | Access Speed | Cleanup Cost | Best For |
|-------|--------------|--------------|--------------|----------|
| **@Singleton** | Once (slow startup) | ⚡ Fastest | None during runtime | Heavy objects |
| **@ActivityRetainedScoped** | Per activity | ⚡ Fast | Low | Session data |
| **@ViewModelScoped** | Per ViewModel | ⚡ Fast | Very low | Screen logic |
| **@ActivityScoped** | Per activity | ⚡ Fast | Low | UI helpers |
| **@FragmentScoped** | Per fragment | ⚡ Fast | Very low | Adapters |
| **@ServiceScoped** | Per service | ⚡ Fast | Medium | Background work |

---

## 🎯 Decision Tree - Visual Guide

```
তোমার dependency কি?

├─ Heavy & Shared (Database, Network)
│  └─ ✅ @Singleton
│
├─ User Session/Flow Data
│  ├─ Rotation survive করবে?
│  │  ├─ YES → @ActivityRetainedScoped
│  │  └─ NO → @ActivityScoped
│  └─
│
├─ Screen-specific Logic
│  ├─ ViewModel এর জন্য?
│  │  └─ ✅ @ViewModelScoped
│  │
│  ├─ Activity UI এর জন্য?
│  │  └─ ✅ @ActivityScoped
│  │
│  └─ Fragment UI এর জন্য?
│     └─ ✅ @FragmentScoped
│
└─ Background Service
   └─ ✅ @ServiceScoped
```

---

## 📱 Real App Example - নাপটিউন App (Story/Lullaby)

| Component | Scope | কেন? | Rotation Impact |
|-----------|-------|------|-----------------|
| **AppDatabase** | `@Singleton` | পুরো app এ 1 DB লাগবে | ✅ Survives |
| **Retrofit API** | `@Singleton` | Network config shared | ✅ Survives |
| **MediaPlayer** | `@Singleton` | App-wide audio player | ✅ Survives |
| **StoryPlaybackSession** | `@ActivityRetainedScoped` | Current story + position preserve | ✅ Survives |
| **StoryListAnalytics** | `@ViewModelScoped` | Per-screen analytics | ✅ Survives |
| **StoryAdapter** | `@FragmentScoped` | Story list adapter | ❌ Recreated |
| **BadgeManager** | `@ActivityScoped` | Activity-wide badge count | ❌ Recreated |
| **AudioService** | `@ServiceScoped` | Background audio playback | N/A |

---

## ⚠️ Common Mistakes Table

| ❌ Wrong Approach | 😱 Problem | ✅ Correct Approach | 💡 Why |
|------------------|-----------|-------------------|--------|
| Database with `@ActivityScoped` | Activity destroy → DB lost | Database with `@Singleton` | DB shared across app |
| Cart with `@Singleton` | User logout → still has old cart | Cart with `@ActivityRetainedScoped` | Activity finish → cart cleared |
| Adapter with `@Singleton` | Memory leak, wrong data shown | Adapter with `@FragmentScoped` | Fragment destroy → adapter cleared |
| Session with `@ActivityScoped` | Rotation → data lost | Session with `@ActivityRetainedScoped` | Rotation → data preserved |
| ViewModel dep with `@Singleton` | All ViewModels share same state | ViewModel dep with `@ViewModelScoped` | Each ViewModel isolated |

---

## 🧠 Quick Reference - Memory Aid

### "SLiMM দিয়ে মনে রাখো"

| Letter | Scope | Mnemonic | Usage |
|--------|-------|----------|-------|
| **S** | **S**ingleton | **S**hared everywhere | Database, Network |
| **L** | Activity**L**ife + Config | **L**asts through rotation | Cart, Session |
| **i** | V**i**ewModel | **i**solated per screen | Screen analytics |
| **M** | Activity **M**anager | **M**anages activity UI | Badge, Navigation |
| **M** | Frag**m**ent | **M**inimal per fragment | Adapter |

**Extra**: **S**ervice = Background **S**ervices

---

## 📊 Scope Hierarchy - Parent-Child

```
Application (Singleton)
    ↓ can inject into
ActivityRetained
    ↓ can inject into
ViewModel
    
Activity (parallel to ActivityRetained)
    ↓ can inject into
Fragment

Service (parallel, separate)
```

**Rule**: Parent scope child scope এ inject করতে পারে, কিন্তু উল্টোটা না।

```kotlin
// ✅ Allowed
@HiltViewModel
class MyViewModel @Inject constructor(
    private val database: AppDatabase  // Singleton → ViewModel ✅
)

// ❌ Not Allowed
@Singleton
class Repository @Inject constructor(
    private val viewModel: MyViewModel  // ViewModel → Singleton ❌ Compile error!
)
```

---

## 🎓 Final Summary Table

| When to Use | Scope Choice |
|-------------|--------------|
| পুরো app জুড়ে | `@Singleton` |
| Activity + rotate survive | `@ActivityRetainedScoped` |
| Screen lifecycle | `@ViewModelScoped` |
| Activity UI coordination | `@ActivityScoped` |
| Fragment UI elements | `@FragmentScoped` |
| Background services | `@ServiceScoped` |

---

**Created by: Hasibuzzaman Chowdhury**  
**Date: December 29, 2025**
