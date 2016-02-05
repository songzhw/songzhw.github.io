# Memory Issue of Kotlin and RxJava

"CleanBeta-Kotlin" (https://github.com/songzhw/cleanBeta-kotlin) is one of my open source project. I try to build a better architecture with ``` Kotlin + RxJava + MVP ``` in this project. The result is pleasant.

However, I find out there are some memory issue about this project. Here is an example. My app just crosses the Splash page, and get to the Home page. I command the SplashActivity to finish itself. But the result is not so pleasant. Here is the memory picture.

![](/imgs/20160202_01.jpg)


Here is the code of SplashActivity
```kotlin

public class SplashActivity : BaseActivity() {
    val jumpObservable : BehaviorSubject<Void> = BehaviorSubject.create()

    var sub1 : Subscription? = null
    var sub2 : Subscription? = null

    var isFinishedSplash = false

    val hello : String by lazy{
        "str"
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_splash)

        // get splash info from server using Retrofit
        var netService = RetrofitSingleton.getNetService()
        sub1 = netService.getSplashInfo_Post(req.msg,req.sign)
                .subscribeOn(Schedulers.newThread())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(
                        { resp : SplashResponse ->
                            var imageLoader = ImageLoader.getInstance()
                            imageLoader.displayImage(resp.splashUrl, ivSplashAd)
                            jumpObservable.onNext(null) // go to the home page
                         },
                        {error ->
                            showToast(error.getMessage().toString())
                        })

        sub2 = jumpObservable.asObservable()
            .delay(2, TimeUnit.SECONDS)
            .subscribe{
                isFinishedSplash = true
                jump(UnlockActivity::class.java)
                this.finish()
            }

    }
}

```


## 1. RxJava

RxJava's life cycle is longer than Activity & Fragment, which means RxJava's object still alive when Activity is finished. Then the Activity object cannot be released by GC. Boom, a memory leak happens.

In the last picture, we can see, the "SplashActivity$onCreate$sub1" is not collected by GC,  


......



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


