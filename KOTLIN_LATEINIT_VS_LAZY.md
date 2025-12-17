# Kotlin এ lateinit vs lazy Properties

এই blog এ আমরা Kotlin এ `lateinit` এবং `lazy` properties সম্পর্কে জানব।

এই দুটি properties আমাদের Kotlin Android project এ খুব ঘন ঘন ব্যবহার করা হয়। আমাদের দুটিকেই খুব ভালোভাবে বুঝতে হবে। আজ আমরা নিম্নলিখিত দুটি সম্পর্কে শিখব:

* `lateinit`
* `lazy`

চলুন `lateinit` property দিয়ে শুরু করি।

## lateinit

Kotlin এ `lateinit` তখন কাজে লাগে যখন আমরা declaration এর সময় একটা variable initialize করতে চাই না এবং পরবর্তীতে কোনো এক সময় initialize করতে চাই, তবে আমরা নিশ্চিত করি যে use করার আগে অবশ্যই initialize করব।

এটি অর্জন করার একটি উপায় (যা ভালো উপায় নয়) হল একটি `nullable` variable তৈরি করা নিচের মতো:
```kotlin
private var mentor: Mentor? = null
```

এবং আমরা সবাই জানি যে Kotlin এ আমাদের যা দরকার তা অর্জন করার জন্য সবসময় একটি ভালো উপায় আছে।

কি হবে যদি আমরা variable টাকে `nullable` বানাতে না চাই?

উত্তর হল `lateinit`।

চলুন আমাদের code update করি:
```kotlin
private lateinit var mentor: Mentor
```

এখানে, আমরা লক্ষ্য করতে পারি যে lateinit keyword আছে এবং variable টি `non-nullable`।

এটাই আমরা চেয়েছিলাম।

এবং পরবর্তীতে আমরা variable টি initialize করতে পারি যখন আমাদের প্রয়োজন হবে নিচের মতো:
```kotlin
fun bookASlot() {
    mentor = Mentor()
}
```

কিন্তু আমাদের অবশ্যই খেয়াল রাখতে হবে যে আমরা variable টি access করার আগে initialize করি, নাহলে এটি নিম্নলিখিত exception throw করবে:
```
kotlin.UninitializedPropertyAccessException: lateinit property mentor has not been initialized
```

Kotlin এ, আমাদের কাছে একটি উপায়ও আছে যাতে check করা যায় যে `lateinit` variable initialize হয়েছে কিনা।

`isInitialized` ব্যবহার করে:
```kotlin
if(this::mentor.isInitialized) {
    // mentor access করুন
} else {
    // অন্য কিছু করুন
}
```

### `lateinit` property ব্যবহার করার সময় বিবেচনা করার বিষয়গুলি:

* শুধুমাত্র `var` keyword এর সাথে ব্যবহার করা যায়।
* শুধুমাত্র `non-nullable` variable এর সাথে ব্যবহার করা যায়।
* ব্যবহার করা উচিত যদি variable mutable হয় এবং পরে initialized করা যায়।
* ব্যবহার করা উচিত যদি আপনি use করার আগে initialization সম্পর্কে নিশ্চিত থাকেন।

এটি ছিল Kotlin এ `lateinit` property সম্পর্কে।

এখন, Kotlin এ `lazy` property সম্পর্কে জানার সময়।

## lazy

Kotlin এ `lazy` তখন কাজে লাগে যখন আমরা একটি class এর ভিতরে একটি object তৈরি করতে চাই, কিন্তু সেই object creation expensive এবং এটি সেই expensive object এর উপর নির্ভরশীল object তৈরিতে delay সৃষ্টি করতে পারে।
```kotlin
class Session {

    private val mentor: Mentor = Mentor()

}
```

ধরুন `Mentor` একটি expensive object। এবং `Session` হল সেই object যা `Mentor` object এর উপর নির্ভরশীল।

যদি `Mentor` object creation তে সময় লাগে, এটি `Session` object তৈরিতে delay করবে।

সুতরাং, এখানেই Kotlin এ `lazy` keyword আমাদের সাহায্য করবে।

চলুন আমাদের code update করি:
```kotlin
class Session {

    private val mentor: Mentor by lazy { Mentor() }

}
```

এখানে, আমরা `lazy` keyword ব্যবহার করেছি।

সুতরাং, আমাদের বুঝতে হবে যে `mentor` object শুধুমাত্র তখনই initialized হবে যখন এটি প্রথমবার access করা হবে, অন্যথায় এটি initialized হবে না।

এটি উপরের উদাহরণে `Session` object এর দ্রুত তৈরি হওয়ার দিকে পরিচালিত করবে কারণ `Mentor` object অপ্রয়োজনীয়ভাবে `Session` এর object creation এর সময় initialized হবে না। এটি প্রথমবার access করার সময় initialized হবে।

### `lazy` property ব্যবহার করার সময় বিবেচনা করার বিষয়গুলি:

* শুধুমাত্র `val` keyword এর সাথে ব্যবহার করা যায়, তাই `read-only` property।
* আমরা চাই variable টি শুধুমাত্র তখনই initialized হোক যখন আমাদের প্রথমবার এটির প্রয়োজন হয়।
* অবশ্যই বুঝতে হবে যে এটি শুধুমাত্র তখনই object তৈরি করে যখন আমরা এটি একদম প্রথমবার access করি এবং তারপরে পরবর্তী access এ, এটি একই object return করে।

এটি ছিল Kotlin এ `lazy` property সম্পর্কে।

এখন আমরা অবশ্যই Kotlin এ `lateinit` vs `lazy` properties বুঝে গেছি।

---

## সারসংক্ষেপ

### lateinit:
- `var` এর সাথে ব্যবহার করতে হবে
- Non-nullable variable এর জন্য
- পরে manually initialize করতে হবে
- ব্যবহারের আগে initialized কিনা check করা যায় (`::property.isInitialized`)

### lazy:
- `val` এর সাথে ব্যবহার করতে হবে
- Read-only property
- প্রথম access এ automatically initialize হয়
- Expensive object creation এর জন্য ideal
- একবার create হলে পরবর্তীতে same object return করে

---

*Last Updated: December 2025*
