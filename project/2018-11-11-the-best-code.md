## What's the best code?
This topic seems too big to be a post. However, I recently think about it a little bit more, so I want to explain my thoguht here for you to consider. 

### 1. Introduction
As a developer, I have a habit to study one open source library every two or three weeks. I just researched [the disposer library](https://github.com/sellmair/disposer) . This is a good library which has some amazing tricks, however, it triggered me to think about what is the best code. 

I want to put the conclusion here at first. I think the best code must have two qualities: 
1). it's easy to be test
2). it's easy to read

Why the testable code would make the code great? Because it's easy to be tested, then it must be separated well. Normally the well-separated code would make your code easy to modify, extend and maintain.

And the quality of good readability is the focus of this post, and also the one that's most likely to be ignored. A lot of dev would mostly focus on the design pattern or architecture pattern when they talk about the good code. Even many famous, great companies are blind to the importance of good readability. I will talk about this part later. 

### 2. Disposable 
In this section, I will mostly talk about the disposer library. This library would help you easily dispose RxJava stream. So you don't have to worry about the memory leak caused by RxJava.

The following code shows an approach to dispose RxJava stream before.

```kotlin
val disposables = CompositableDisposable()
disposables.add(
    Service.queryAwesomeData()
         .subscribe()
)
```

This is okay, but just not that elegant, because this is not a chaining calls, which made it hard to read. Here is how the disposable library change your code:
    
```kotlin
val disposer: Disposer = /* ... */
Service.queryAwesomeData().subscribe().disposeBy(disposer) // Managed by the disposer
```


