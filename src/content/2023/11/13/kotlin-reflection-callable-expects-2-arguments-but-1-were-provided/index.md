---
title: "Kotlin - Reflection - Callable expects 2 arguments, but 1 were provided."
date: 2023-11-13
categories: 
  - "kotlin"
image: "/images/image-5.png"
---

When using reflection in Kotlin, this problem arises because you try to call a function without sendind the instance class as parameter.

```kotlin
class ImAClass() {
    fun function(param: Int): Int {
        println("Function called, params: $param ")
        return param
    }
}
```

If you call the function like this

```kotlin
ImAClass::function.call(1)
```

You will get an exception like this

Exception in thread "main" java.lang.IllegalArgumentException: Callable expects 2 arguments, but 1 were provided.  
at kotlin.reflect.jvm.internal.calls.Caller$DefaultImpls.checkArguments(Caller.kt:20)  
at kotlin.reflect.jvm.internal.calls.CallerImpl.checkArguments(CallerImpl.kt:15)  
at kotlin.reflect.jvm.internal.calls.CallerImpl$Method$Instance.call(CallerImpl.kt:112)  
at kotlin.reflect.jvm.internal.KCallableImpl.call(KCallableImpl.kt:108)  
at kotlin.jvm.internal.CallableReference.call(CallableReference.java:161)  
at MainKt.main(Main.kt:156)  
at MainKt.main(Main.kt)

So, you have to send an instance of the class first.

```kotlin
ImAClass::function.call(ImAClass(), 1)
```

![](/images/image-5.png)
