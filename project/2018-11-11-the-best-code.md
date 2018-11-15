# What's the best code?
This topic seems too big to be a post. However, I recently think about it a little bit more, so I want to explain my thoguht here for you to consider. 

## 1. Introduction
As a developer, I have a habit to study one open source library every two or three weeks. I just researched [the disposer library](https://github.com/sellmair/disposer) . This is a good library which has some amazing tricks, however, it triggered me to think about what is the best code. 

I want to put the conclusion here at first. I think the best code must have two qualities: 
1). it's easy to be test
2). it's easy to read

Why the testable code would make the code great? Because it's easy to be tested, then it must be separated well. Normally the well-separated code would make your code easy to modify, extend and maintain.

And the quality of good readability is the focus of this post, and also the one that's most likely to be ignored. A lot of dev would mostly focus on the design pattern or architecture pattern when they talk about the good code. Even many famous, great companies are blind to the importance of good readability. I will talk about this part later. 

## 2. Disposable 

### 2.1 How to use Disposable
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

class MainActivity : AppCompatActivity() {
     fun foo() {
        DummyNetwork.query()
            .subscribe(::displayResponse)
            .disposeBy(onStop) // "disposeBy()" is the key part!
    }

    private fun displayResponse(response: String) {
        textView.text = response
    }
}
```

The above is the pros. Now let's consider what's the cons. This library was very easy to use, however it's hard to read its own code.

### 2.2 Dive into the code
The key code is the `disposeBy(onStop)` method. We have two questions: 1). how the disposeBy() works;  2). what is this onStop?

#### 2.2.1 Getting lost
Now let's take a look at the code:
```kotlin
// so the onStop is the extension property of LifecycleOwner 
// Our AppcompatActivity is  a child of LifecycleOwner
val LifecycleOwner.onStop get() = lifecycle.onStop
```
lifecyle is obvious from `LifecycleOwner.getLifecycle()`. But what is lifecycle.onStop? Let's continue to dig:

```kotlin
val Lifecycle.onStop get() = disposers.onStop

// what is disposers then? keep digging
val Lifecycle.disposers: LifecycleDisposers get() = LifecycleDisposers.Store[this]

// what is LifecycleDisposers.Store[this] ?
class LifecycleDisposers(
    val onCreate: Disposer,
    val onStop: Disposer,
    ...    
    internal object Factory    
    internal object Store
}

```

Dear reader, I believe that you must be lost when reading the code. I was when I read this code. I was lost, Store is obviously an object. But if I was right, I thought I saw `Store[this]` ?! What is this?

#### 2.2.2 Override Operator
It took a while to find out what the `Store[this]` means because the IDE, Android Studio, does not support the navigation to the declare when clicking the `[]`.

Yes, this `[]` is an overrided operator.

```kotlin
// operator get(): means you are custom the `[]` operator
internal operator fun LifecycleDisposers.Store.get(lifecycle: Lifecycle): LifecycleDisposers =
        disposersLock.withLock {
            disposers.getOrPut(lifecycle) {
                LifecycleDisposers.Factory.create(lifecycle)
            }
        }
```


### 2.3 Retro
Okay, although you don't know the whole code in the disposable library, you must have a feeling what the code looks like.

Yes, I have the same feeling: it's a great library, but it's really pretty hard to read it through.

The most two primary challenges for those who want to read the code are: 
1). It uses a lot of extension, a lot of which are from a huge chain of extensions. The previous code actually comes from multiple files, which makes the read more difficult.
2). It override some operator. Since the IDE does not "go to declare", it would hard for people to trace some issue, read the code and find the correct code.

In short, this library is brillian. However, its own source code is a disaster to look at, in my opinion.

## 3. More examples
Disposable library is not the only one that has the awful code to read. I could list a lot in this post. Here, I just take the take two examples: Fragment and Dagger.

I know this is extremly controversial. However, I want to write my thought down to discuss with my readers. If you have different opinion, you can email me: <songzhw2012@gmail.com>

### 3.1 Fragment
Fragment was invented since Android 3.0 in 2011. It's been almost 8 years since then. I was happy to see it at first, I thought this could make our Views as a part of big screen. This way, we could reuse these specific parts. Also, it's good to see Fragment can make our layout be easy to extend between Phone and Tablet.

A few years later. My disappointmet at Fragment are growing. At last, I don't want use it in most case. (DialogFragment may be the exception.)

The reasons why I am so disappointed are as follows:
1). Fragment does not save the Activity to be a God Class. Lots of dev still write business code and ui code in the same Fragment class. 
2). View could be reuseable. Why bother to use Fragment in many cases?
3). Fragment still has some feature that I'm not confortable with, such as the "famous" error - "can not perform this action after onsaveinstancestate".
4). I am a guy who like to make tools to make our dev life easier. In the past time, I only need to make some global tools just for Activity. Now I had to do the same trick to Fragments. Normally it's not that easy. To be honest, even Google has faced the same issue. The prematural lifecycle library in Architecture Compnent libraries can only suppport Activity first. Like half year later, Google then had finally succeed to support Fragment. This is what I mean. Activity + Fragment just doubled your work if you are making some great tools for global features.

Buth the above argument is not about this post. This post I want to make a point about how to make a great code. Now since Fragment has obviously duplicated the same job of Activity (they both has lifecycle, could refresh UI and business logic), the code is not elegant anymore. Let's say you have an Activity contains one Fragment. You need to find out which one single view belong to, Activity or Fragment? Same logic apply to the logic code. So the readablity is now uncertain. Some code could be in the Fragment, or Activity. You may need to jump from these two files back and froth to get to understand a single work flow.

### 3.2 Dagger
Dagger is another library that makes me excited at first, and then hurt me deeply later. 




### 3.2 Dagger




