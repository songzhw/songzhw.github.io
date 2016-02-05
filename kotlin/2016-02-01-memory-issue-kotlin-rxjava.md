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

In the last picture, we can see, the "SplashActivity$onCreate$sub1$2" is not collected by GC.



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
Coming back to the previous picture, I noticed that even I use ``` sub1.unsubscribe()``` , and I still have some objects that is released with the ```splashActivity.finish()```

So I start to suspect that kotlin maybe the reason. 

Back to Java, as we know , annoynmous nestted class in Activity actually holds a reference of this container Activity. This means when the Activity want to finish, it may not succeed. 

``` java
public class OneActivity extends Activity{

    public void onCreate(Bundle b){
        Observable.just(12)
            .subscribe(
                new Actino1<Integer>(){
                    @Override
                    public void call(Integer i){
                        System.out.print("result = "+i);
                    }
                }
                );
    }
}
```
In previous codes, the ```new Action1()``` is a annoynmous class in the Activity. When the RxJava's lifecycle is longer than Activity's, this is a trap for our memory.

The same code in kotlin is like this:

``` kotlin
public class SplashActivity : BaseActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        Observable.just(12)
            .subscribe{ println("result = $it")}
    }

}
```
As we know, kotlin has some beatiful syntax sugar, and closure is one of them. The kotlin code is a obvious example compared to the same java code, right? 

However, what the real implemention of kotlin closure in the JVM? Was it an annoynmous nestted class? If it is yes, we may have the same problem. 

Let's see the memory leak picture again.
 
 ![](/imgs/20160202_01.jpg)

I have two suspetions:

1. lambda

the ```SplashActivity$onCreate$sub1$2``` is like an annoynmous netted class. And the previous reason may cause the memory leak.

2. lazy member

the ```SplashActivity$hello$1``` and ``` SplashActivity$hello$2``` are not released neither. What's the two stuff?

Let's get back to the Splash code

```kotlin
    val hello : String by lazy{
        "str"
    }
```

what does this ```lazy``` keyword mean? Jetbrains tells us:

```
lazy() is a function that takes a lambda and returns an instance of Lazy<T> which can serve as a delegate for implementing a lazy property: the first call to get() executes the lambda passed to lazy() and remembers the result, subsequent calls to get() simply return the remembered result.
```

We usually use ```lazy``` keyword to postpone the initialization of some resource, like this

``` kotlin
private val columns by lazy { 
    resources.getInteger(R.integer.num_columns) 
}

```

Back to our memory leak, we noticed our ```hello``` member have two leaks. In my opiinion, every lazy memeber may have two objects in it, maybe getter and setter , or maybe getter and recorder (to record this memeber is initilzed or not), or maybe something and somehting. Anyway, its two object maybe holds the Activity and do not release it when the Activity is about ot finish. 

## 3. Conclusion
Right now, I do have no solution about Kotlin and RxJava. Maybe RxLifeCycle is a helper, but kotlin is a two-blade dagger, which is really useful and also may be harmful too. If I have soultion later, I will post it again.

