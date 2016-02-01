# Memory Issue of Kotlin and RxJava

"CleanBeta-Kotlin" (https://github.com/songzhw/cleanBeta-kotlin) is one of my open source project. I try to build a better architecture with ``` Kotlin + RxJava + MVP ``` in this project. The result is pleasant.

However, I find out there are some memory issue about this project. Here is an example. My app just crosses the Splash page, and get to the Home page. I command the SplashActivity to finish itself. But the result is not so pleasant. Here is the memory picture.

![](/imgs/20160202_01.jpg)


I think Kotlin and RxJava are both have some responsibility.

## 1. RxJava

RxJava's life cycle is longer than Activity & Fragment, which means RxJava's object still alive when Activity is finished. Then the Activity object cannot be released by GC. Boom, a memory leak happens.

Here is some tips about RxJava:
1. do not use 

```kotlin
	Observable.just(12)
		.subscribe{ ... }
```

Instead, use 

```kotlin
	var subscription = Observable.just(12)
		.subscribe{ ... }

	// in the onDestory() or onStop()
	subscription.unsuscribe(); // release the reference for GC
```

2. If you have too many subscription, ```CompositeSubscription``` is here to rescue you. 

```CompositeSubscription``` is like a List<Subscriptoin>.  

You call ``` compositeSubscription.add(oneSubscription) ``` to let it remember every subscription. 

Then you only call ``` compositeSubscription.unsubscribe() ``` in the Activity.onDestory(), every subscription remembered by CompositeSubscription will be all unsubscribed.



3. you may use RxLifeCycle to control your RxJava's life cycle.

 RxLifeCycle is a library that helps you to handle the RxJava's life cycle to avoid the memory leak. 

 The specific usage of this library is here :  https://github.com/trello/RxLifecycle



## 2. Kotlin


