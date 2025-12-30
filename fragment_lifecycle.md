# 📱 Fragment Lifecycle - বিস্তারিত বাংলা গাইড

## 🤔 Fragment Lifecycle কেন বেশি জটিল?

### Activity vs Fragment:
```
Activity Lifecycle: 7 methods
Fragment Lifecycle: 11 methods! 😱

কেন বেশি?
- Fragment একা থাকতে পারে না
- Activity এর সাথে attached থাকতে হয়
- View আলাদাভাবে create/destroy হয়
- Fragment বেঁচে থাকতে পারে কিন্তু View destroyed!
```

---

## 📊 Complete Fragment Lifecycle Diagram

```
Activity started
       ↓
┌──────────────┐
│ onAttach()   │  ← Activity তে যুক্ত হলো
└──────────────┘
       ↓
┌──────────────┐
│ onCreate()   │  ← Fragment created (Activity এর মতো)
└──────────────┘
       ↓
┌──────────────┐
│onCreateView()│  ← 🌟 UI তৈরি করো (Fragment specific!)
└──────────────┘
       ↓
┌──────────────┐
│onViewCreated()│ ← 🌟 View ready, setup করো (Important!)
└──────────────┘
       ↓
┌──────────────┐
│ onStart()    │  ← Visible হচ্ছে
└──────────────┘
       ↓
┌──────────────┐
│ onResume()   │  ← Active! User interact করতে পারে
└──────────────┘
       ↓
   ╔══════════╗
   ║ RUNNING  ║  ← Fragment fully active!
   ╚══════════╝
       ↓
┌──────────────┐
│ onPause()    │  ← Paused (Activity এর মতো)
└──────────────┘
       ↓
┌──────────────┐
│ onStop()     │  ← Stopped (Invisible)
└──────────────┘
       ↓
┌──────────────┐
│onDestroyView()│ ← 🌟 View destroy! (Fragment alive!)
└──────────────┘
       ↓
┌──────────────┐
│ onDestroy()  │  ← Fragment destroy
└──────────────┘
       ↓
┌──────────────┐
│ onDetach()   │  ← Activity থেকে আলাদা হলো
└──────────────┘
       ↓
    (Dead 💀)
```

---

## 🎓 প্রতিটা Method বিস্তারিত

### 1️⃣ onAttach(context: Context)

**কখন call হয়:**
```
✅ Fragment যখন Activity তে attach হয়
✅ সবার আগে call হয়
✅ Activity এর reference পাওয়া যায়
```

**কি করতে হয়:**
```kotlin
override fun onAttach(context: Context) {
    super.onAttach(context)
    
    // Activity থেকে interface পাও
    if (context is OnDataPassListener) {
        listener = context
    }
    
    // Activity cast করতে পারো (if needed)
    activity?.let {
        // Activity reference পেয়ে গেছি
    }
    
    Log.d("Fragment", "onAttach: Fragment attached to activity")
}
```

**Real Example:**
```kotlin
// Interface for communication
interface OnUserSelectedListener {
    fun onUserSelected(userId: String)
}

class UserListFragment : Fragment() {
    
    private var listener: OnUserSelectedListener? = null
    
    override fun onAttach(context: Context) {
        super.onAttach(context)
        
        // Activity must implement interface
        if (context is OnUserSelectedListener) {
            listener = context
        } else {
            throw RuntimeException("$context must implement OnUserSelectedListener")
        }
        
        Log.d("UserListFragment", "Attached to activity")
    }
    
    private fun selectUser(userId: String) {
        // Activity কে notify করো
        listener?.onUserSelected(userId)
    }
}
```

---

### 2️⃣ onCreate(savedInstanceState: Bundle?)

**কখন call হয়:**
```
✅ Fragment তৈরি হওয়ার সময়
✅ Activity এর onCreate এর মতো
✅ UI তৈরি হয়নি এখনো!
```

**কি করতে হয়:**
```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    
    // Arguments receive করো
    val userId = arguments?.getString("USER_ID")
    
    // Non-UI initialization
    // ViewModel initialize করতে পারো
    viewModel = ViewModelProvider(this)[UserViewModel::class.java]
    
    // Retained instance (optional)
    retainInstance = true
    
    // Options menu (if needed)
    setHasOptionsMenu(true)
    
    Log.d("Fragment", "onCreate: Fragment created")
}
```

**Important Notes:**
```
⚠️ View পাবে না এখানে!
⚠️ findViewById() call করো না!
⚠️ শুধু non-UI initialization
```

---

### 3️⃣ onCreateView() - 🌟 MOST IMPORTANT!

**কখন call হয়:**
```
✅ Fragment এর UI তৈরি করার সময়
✅ Layout inflate করার জন্য
✅ View return করতে হবে
```

**কি করতে হয়:**
```kotlin
override fun onCreateView(
    inflater: LayoutInflater,
    container: ViewGroup?,
    savedInstanceState: Bundle?
): View? {
    // Method 1: Traditional inflate
    return inflater.inflate(R.layout.fragment_user, container, false)
    
    // Method 2: View Binding (Recommended!)
    _binding = FragmentUserBinding.inflate(inflater, container, false)
    return binding.root
}
```

**Complete Example with View Binding:**
```kotlin
class UserFragment : Fragment() {
    
    private var _binding: FragmentUserBinding? = null
    private val binding get() = _binding!!
    
    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View {
        _binding = FragmentUserBinding.inflate(inflater, container, false)
        
        Log.d("UserFragment", "onCreateView: View inflated")
        
        return binding.root
    }
}
```

**Critical Rules:**
```
⚠️ MUST return a View!
⚠️ container parameter use করো inflate এ
⚠️ attachToRoot = false (ALWAYS!)
⚠️ View setup করো না এখানে, onViewCreated() এ করো!
```

**Why attachToRoot = false?**
```kotlin
// ❌ WRONG
inflater.inflate(R.layout.fragment_user, container, true)
// container এ automatically add হয়ে যাবে
// Fragment system এটা নিজে করে!
// Crash হবে! 😱

// ✅ CORRECT
inflater.inflate(R.layout.fragment_user, container, false)
// Fragment system add করবে proper timing এ
```

---

### 4️⃣ onViewCreated() - 🌟 SETUP করার জায়গা!

**কখন call হয়:**
```
✅ onCreateView() এর পরে
✅ View তৈরি হয়ে গেছে
✅ findViewById() safe এখন!
```

**কি করতে হয়:**
```kotlin
override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
    super.onViewCreated(view, savedInstanceState)
    
    // 1. View setup করো
    setupViews()
    
    // 2. Click listeners
    setupListeners()
    
    // 3. RecyclerView setup
    setupRecyclerView()
    
    // 4. ViewModel observe করো
    observeData()
    
    // 5. Load data
    loadData()
    
    Log.d("Fragment", "onViewCreated: View ready!")
}
```

**Complete Real Example:**
```kotlin
class UserListFragment : Fragment() {
    
    private var _binding: FragmentUserListBinding? = null
    private val binding get() = _binding!!
    
    private val viewModel: UserViewModel by viewModels()
    private lateinit var userAdapter: UserAdapter
    
    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View {
        _binding = FragmentUserListBinding.inflate(inflater, container, false)
        return binding.root
    }
    
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        
        setupRecyclerView()
        setupListeners()
        observeData()
        
        // Load data
        viewModel.loadUsers()
    }
    
    private fun setupRecyclerView() {
        userAdapter = UserAdapter { user ->
            // Handle click
            navigateToDetail(user.id)
        }
        
        binding.recyclerView.apply {
            layoutManager = LinearLayoutManager(requireContext())
            adapter = userAdapter
        }
    }
    
    private fun setupListeners() {
        binding.fabAdd.setOnClickListener {
            // Add new user
            showAddUserDialog()
        }
        
        binding.swipeRefresh.setOnRefreshListener {
            viewModel.refreshUsers()
        }
    }
    
    private fun observeData() {
        // 🌟 viewLifecycleOwner use করো!
        viewModel.users.observe(viewLifecycleOwner) { users ->
            userAdapter.submitList(users)
            binding.swipeRefresh.isRefreshing = false
        }
        
        viewModel.error.observe(viewLifecycleOwner) { error ->
            Toast.makeText(requireContext(), error, Toast.LENGTH_SHORT).show()
        }
    }
    
    private fun navigateToDetail(userId: String) {
        val action = UserListFragmentDirections.actionToDetail(userId)
        findNavController().navigate(action)
    }
}
```

**onCreateView vs onViewCreated:**
```
┌─────────────────┬──────────────────────┐
│  onCreateView() │  onViewCreated()     │
├─────────────────┼──────────────────────┤
│ Inflate layout  │ Setup views          │
│ Return view     │ Set listeners        │
│ NO setup!       │ Observe data         │
│                 │ Initialize adapters  │
└─────────────────┴──────────────────────┘
```

---

### 5️⃣ onStart() - Visible হচ্ছে

**কখন call হয়:**
```
✅ Fragment visible হচ্ছে
✅ Activity এর onStart এর সাথে
```

**কি করতে হয়:**
```kotlin
override fun onStart() {
    super.onStart()
    
    // Animation শুরু করতে পারো
    startAnimations()
    
    // UI updates
    updateUI()
    
    Log.d("Fragment", "onStart: Fragment becoming visible")
}
```

---

### 6️⃣ onResume() - Active!

**কখন call হয়:**
```
✅ Fragment fully interactive
✅ User interact করতে পারে
✅ Activity এর onResume এর সাথে
```

**কি করতে হয়:**
```kotlin
override fun onResume() {
    super.onResume()
    
    // Camera start করো
    cameraManager?.startCamera()
    
    // Location updates
    locationManager?.requestLocationUpdates()
    
    // Analytics tracking
    analyticsTracker.trackScreenView("UserListFragment")
    
    Log.d("Fragment", "onResume: Fragment active!")
}
```

**Real Example:**
```kotlin
class CameraFragment : Fragment() {
    
    private var cameraManager: CameraManager? = null
    
    override fun onResume() {
        super.onResume()
        
        // Start camera when visible
        if (hasCameraPermission()) {
            cameraManager?.startCamera()
            Log.d("CameraFragment", "Camera started")
        }
    }
    
    override fun onPause() {
        super.onPause()
        
        // Stop camera when not visible
        cameraManager?.stopCamera()
        Log.d("CameraFragment", "Camera stopped")
    }
}
```

---

### 7️⃣ onPause() - Pausing

**কখন call হয়:**
```
✅ Fragment losing focus
✅ Another fragment দেখানো হচ্ছে
✅ Activity paused
```

**কি করতে হয়:**
```kotlin
override fun onPause() {
    super.onPause()
    
    // Media pause করো
    mediaPlayer?.pause()
    
    // Quick save
    saveFormData()
    
    // Camera stop
    cameraManager?.stopCamera()
    
    Log.d("Fragment", "onPause: Fragment paused")
}
```

**Rule: MUST BE FAST!**
```
⚠️ onPause() must complete quickly!
⚠️ Heavy operations করো না
⚠️ Database save এখানে না, onStop এ করো
```

---

### 8️⃣ onStop() - Not Visible

**কখন call হয়:**
```
✅ Fragment completely hidden
✅ User navigate করে অন্য screen এ গেছে
✅ Activity stopped
```

**কি করতে হয়:**
```kotlin
override fun onStop() {
    super.onStop()
    
    // Database save করতে পারো
    lifecycleScope.launch {
        saveToDatabase()
    }
    
    // Network calls cancel করো
    cancelPendingRequests()
    
    // Broadcast receiver unregister
    requireContext().unregisterReceiver(myReceiver)
    
    Log.d("Fragment", "onStop: Fragment hidden")
}
```

---

### 9️⃣ onDestroyView() - 🌟 CRITICAL!

**কখন call হয়:**
```
✅ View destroy হচ্ছে
✅ Fragment replace/remove করলে
✅ Back stack থেকে pop করলে
✅ কিন্তু Fragment এখনো alive! (Important!)
```

**কি করতে হয়:**
```kotlin
override fun onDestroyView() {
    super.onDestroyView()
    
    // 🌟 MUST: Binding cleanup!
    _binding = null
    
    // View references null করো
    adapter = null
    recyclerView = null
    
    Log.d("Fragment", "onDestroyView: View destroyed")
}
```

**Why so important?**
```
Fragment থাকতে পারে কিন্তু View destroy হতে পারে!

Scenario: Fragment back stack এ আছে
         ↓
View destroy হয়ে যায় (onDestroyView)
         ↓
Fragment বেঁচে আছে! (onDestroy NOT called!)
         ↓
_binding reference ধরে রাখলে = MEMORY LEAK! 😱
```

**Complete Example:**
```kotlin
class UserFragment : Fragment() {
    
    private var _binding: FragmentUserBinding? = null
    private val binding get() = _binding!!
    
    private var adapter: UserAdapter? = null
    
    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View {
        _binding = FragmentUserBinding.inflate(inflater, container, false)
        return binding.root
    }
    
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        
        adapter = UserAdapter()
        binding.recyclerView.adapter = adapter
    }
    
    override fun onDestroyView() {
        super.onDestroyView()
        
        // 🌟 Cleanup!
        _binding = null  // Memory leak prevention!
        adapter = null
        
        Log.d("UserFragment", "View destroyed, references cleaned")
    }
}
```

---

### 🔟 onDestroy() - Fragment শেষ

**কখন call হয়:**
```
✅ Fragment completely destroyed
✅ Activity finishing
✅ Fragment removed permanently
```

**কি করতে হয়:**
```kotlin
override fun onDestroy() {
    super.onDestroy()
    
    // Final cleanup
    viewModel.clearData()
    
    // Cancel coroutines
    job?.cancel()
    
    Log.d("Fragment", "onDestroy: Fragment destroyed")
}
```

---

### 1️⃣1️⃣ onDetach() - Activity থেকে আলাদা

**কখন call হয়:**
```
✅ Fragment Activity থেকে detach হচ্ছে
✅ শেষ callback
```

**কি করতে হয়:**
```kotlin
override fun onDetach() {
    super.onDetach()
    
    // Activity reference clear করো
    listener = null
    
    Log.d("Fragment", "onDetach: Detached from activity")
}
```

---

## 🎯 Real-World Scenarios

### Scenario 1: Fragment Replace (Back Stack এ push)

```
Current Fragment (A)
       ↓
Replace with Fragment B
       ↓
Fragment A:
   onPause()
   onStop()
   onDestroyView()   ← View destroy! Fragment alive!
       ↓
Fragment B:
   onCreate()
   onCreateView()
   onViewCreated()
   onStart()
   onResume()
       ↓
   (Fragment B active, Fragment A in back stack)
       ↓
Back Button Press
       ↓
Fragment B:
   onPause()
   onStop()
   onDestroyView()
   onDestroy()
   onDetach()
       ↓
Fragment A:
   onCreateView()    ← View recreate!
   onViewCreated()
   onStart()
   onResume()
```

**Code Example:**
```kotlin
// MainActivity
fun navigateToDetail() {
    supportFragmentManager.beginTransaction()
        .replace(R.id.container, DetailFragment())
        .addToBackStack(null)  // Back stack এ রাখলাম
        .commit()
    
    // ListFragment:
    // - onDestroyView() called (View gone!)
    // - Fragment still alive!
    
    // Back button:
    // - onCreateView() called again (View recreate!)
}
```

---

### Scenario 2: Fragment Replace (No Back Stack)

```
Fragment A active
       ↓
Replace with Fragment B (no back stack)
       ↓
Fragment A:
   onPause()
   onStop()
   onDestroyView()
   onDestroy()      ← Completely destroyed!
   onDetach()
       ↓
Fragment B:
   onAttach()
   onCreate()
   onCreateView()
   onViewCreated()
   onStart()
   onResume()
```

---

### Scenario 3: Activity Configuration Change (Screen Rotate)

```
Fragment active
       ↓
Screen Rotate
       ↓
Activity destroyed:
Fragment:
   onPause()
   onStop()
   onDestroyView()
   onDestroy()
   onDetach()
       ↓
Activity recreated:
Fragment:
   onAttach()
   onCreate()
   onCreateView()
   onViewCreated()
   onStart()
   onResume()
```

---

## 🎯 viewLifecycleOwner vs lifecycleOwner

### এটা Fragment এর সবচেয়ে confusing part!

```kotlin
Fragment এর 2টা Lifecycle আছে:

1. Fragment Lifecycle (this)
   - onCreate to onDestroy
   
2. View Lifecycle (viewLifecycleOwner)
   - onCreateView to onDestroyView
```

### Why 2 Lifecycles?

```
Fragment back stack এ যেতে পারে:
   ↓
View destroy হয় (onDestroyView)
   ↓
কিন্তু Fragment alive!
   ↓
ফিরে এলে View আবার create হয় (onCreateView)
```

### Which One to Use?

```kotlin
// ❌ WRONG - Fragment lifecycle use করছি
class MyFragment : Fragment() {
    
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        
        viewModel.data.observe(this) { data ->  // ❌ this = Fragment
            binding.textView.text = data
            // Fragment alive থাকলে observe চলতে থাকবে
            // কিন্তু View destroy হয়ে গেছে!
            // binding.textView = null! 😱
            // CRASH!
        }
    }
}

// ✅ CORRECT - View lifecycle use করছি
class MyFragment : Fragment() {
    
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        
        viewModel.data.observe(viewLifecycleOwner) { data ->  // ✅
            binding.textView.text = data
            // View destroy হলে observe stop!
            // Safe! ✅
        }
    }
}
```

### Rule of Thumb:
```
onViewCreated() এর ভিতরে:
   ↓
ALWAYS use viewLifecycleOwner
   ↓
View এর সাথে related সব কিছুতে
```

---

## 📊 Lifecycle Methods Comparison

| Method | Activity Equivalent | UI Access | Common Use |
|--------|-------------------|-----------|------------|
| **onAttach** | - | ❌ NO | Get Activity reference |
| **onCreate** | onCreate | ❌ NO | Non-UI initialization |
| **onCreateView** | - | ❌ NO | Inflate layout |
| **onViewCreated** | - | ✅ YES | Setup UI, listeners |
| **onStart** | onStart | ✅ YES | Animations |
| **onResume** | onResume | ✅ YES | Camera, sensors |
| **onPause** | onPause | ✅ YES | Pause media |
| **onStop** | onStop | ✅ YES | Save data |
| **onDestroyView** | - | ❌ NO | Cleanup binding! |
| **onDestroy** | onDestroy | ❌ NO | Final cleanup |
| **onDetach** | - | ❌ NO | Remove references |

---

## ⚠️ Common Mistakes - Summary

### 1. Binding না ছাড়া
```kotlin
// ❌ Memory leak!
private lateinit var binding: FragmentUserBinding

// ✅ Correct
private var _binding: FragmentUserBinding? = null
private val binding get() = _binding!!

override fun onDestroyView() {
    super.onDestroyView()
    _binding = null  // MUST!
}
```

### 2. Wrong lifecycle owner
```kotlin
// ❌ Fragment lifecycle
viewModel.data.observe(this) { }

// ✅ View lifecycle
viewModel.data.observe(viewLifecycleOwner) { }
```

### 3. View setup wrong place
```kotlin
// ❌ onCreate - View নেই এখনো!
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    binding.textView.text = "Hello"  // Crash! 😱
}

// ✅ onViewCreated - View ready!
override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
    super.onViewCreated(view, savedInstanceState)
    binding.textView.text = "Hello"  // Safe! ✅
}
```

---

## 🎯 Best Practices Checklist

```
✅ onCreateView - শুধু inflate
✅ onViewCreated - সব setup এখানে
✅ viewLifecycleOwner - observe করার সময়
✅ onDestroyView - _binding = null (MUST!)
✅ requireContext() - context পাওয়ার জন্য
✅ Safe Args - type-safe arguments
✅ Navigation Component - modern navigation
```

---

## 💡 Memory Trick

```
Fragment Lifecycle = 2 Parts

Part 1: Fragment নিজে
onCreate → onDestroy

Part 2: Fragment এর View
onCreateView → onDestroyView

View আগে মরতে পারে Fragment এর আগে!
তাই cleanup করো onDestroyView তে!
```

---

## 🎓 Interview Questions

**Q: Fragment lifecycle Activity থেকে বেশি complex কেন?**
```
A: Fragment এর 2টা lifecycle আছে:
1. Fragment lifecycle (onCreate to onDestroy)
2. View lifecycle (onCreateView to onDestroyView)

Fragment back stack এ থাকতে পারে View ছাড়া।
```

**Q: viewLifecycleOwner কি এবং কেন ব্যবহার করবে?**
```
A: Fragment এর View এর lifecycle owner।
onViewCreated এ observe করার সময় use করতে হয়।
View destroy হলে automatically observe stop হয়।
Memory leak prevention!
```

**Q: onCreateView vs onViewCreated পার্থক্য?**
```
A:
onCreateView:
- Layout inflate করো
- View return করো
- Setup করো না!

onViewCreated:
- View setup করো
- Listeners set করো
- Data observe করো
```

**Q: onDestroyView এ _binding = null কেন?**
```
A: Fragment back stack এ থাকতে পারে।
View destroy হয় কিন্তু Fragment alive।
_binding reference থাকলে memory leak!
```

---

## 📊 Quick Reference

### Fragment Lifecycle Flow:
```
Attach → Create → CreateView → ViewCreated → Start → Resume
   ↓
ACTIVE
   ↓
Pause → Stop → DestroyView → Destroy → Detach
```

### Critical Cleanup Points:
```
onPause:     Quick saves only
onStop:      Database saves
onDestroyView: _binding = null (MUST!)
onDestroy:   Final cleanup
onDetach:    listener = null
```

---

**সারাংশ: Fragment lifecycle Activity এর চেয়ে complex কারণ Fragment এর View আলাদাভাবে lifecycle আছে। viewLifecycleOwner এবং onDestroyView cleanup master করলে তুমি Fragment expert! 🚀**

**Created by: Claude for Hasibuzzaman Chowdhury**  
**Date: December 30, 2025**
