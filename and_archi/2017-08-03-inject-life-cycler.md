## Introduction
Consider, for example, the requirement that asked you to register location SDK on activity.onResume(), and, of course, unregister it on activity.onPause(). You may just add these two register and unregister code to every Activty that need to detect the location change. But this is ugly, and there are duplicate code.

Injecting the activity's lifecycle is a good solution. So you don't have to add the same logic in every activity that needs to know the change of location. 

But how could we inject the activity's lifecycle. Normally there are three ways to do that. In this post I will introduce three ways to implement this.

## 01. Application.ActivityLifecycleCallbacks
If your minSDK is equal to or larger than 14, you can use Application.ActivityLifecycleCallbacks.

The code below shows how to do that
```java
public class YourApplication extends Application {

    @Override
    public void onCreate() {  
      super.onCreate();  
      this.registerActivityLifecycleCallbacks(new ActivityLifecycleCallbacks() {  
       
            @Override  
            public void onActivityStopped(Activity activity) {  
                Logger.v(activity, "onActivityStopped");  
            }  
           
            @Override  
            public void onActivityStarted(Activity activity) {  
                Logger.v(activity, "onActivityStarted");  
            }  
           
            @Override  
            public void onActivitySaveInstanceState(Activity activity, Bundle outState) {  
                Logger.v(activity, "onActivitySaveInstanceState");  
            }  
           
            @Override  
            public void onActivityResumed(Activity activity) {  
                Logger.v(activity, "onActivityResumed");  
            }  
           
            @Override  
            public void onActivityPaused(Activity activity) {  
                Logger.v(activity, "onActivityPaused");  
            }  
           
            @Override  
            public void onActivityDestroyed(Activity activity) {  
                Logger.v(activity, "onActivityDestroyed");  
            }  
           
            @Override  
            public void onActivityCreated(Activity activity, Bundle savedInstanceState) {  
                Logger.v(activity, "onActivityCreated");  
        }  
          });  
    }
}
```

This approach apparently will detect all the life cycle change of all Activities.  The beauty of it is you can get the instance of Activity, which means you can inject a presenter or other decoupled object to this activity. Without it, it would be very hard to inject a presenter to one activity since the creation of one Activity is done by Android OS, not ourselves.

Of course, the cost of this approach is
* it will detect all the activities. If you just want to listen to soem particular activities, this approach may seem a little heavy for you.
* it does not support Fragment

## 02. LifecycleActivity
Google I/O 2017 release an Android Architecture Component library (~_~, loooong name). There is one lifecycle sublibrary among it. We can take advantage of it to hijack the lifecycle methods.

Here is what we do:
1. project/build.gradle
```groovy
allprojects {
    repositories {
        ...
        maven { url 'https://maven.google.com' }
    }
}
```

2. project/app/build.gradle
```groovy
    compile "android.arch.lifecycle:runtime:1.0.0-alpha5"
    compile "android.arch.lifecycle:extensions:1.0.0-alpha5"
    annotationProcessor "android.arch.lifecycle:compiler:1.0.0-alpha5"

```

3. Add a observer that want to know when lifecycle changes
```java
public class Watcher01 implements LifecycleObserver {
    @OnLifecycleEvent(Lifecycle.Event.ON_ANY)
    public void onAny(LifecycleOwner owner, Lifecycle.Event event) {
        System.out.println("szw watcher : onAny(" + event.name() + ")" + " ; owner = " + owner);
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_RESUME)
    public void onResume() {
        System.out.println("szw watcher : onResume()");
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_PAUSE)
    public void onPause() {
        System.out.println("szw watcher : onPause()");
    }
}
```

4. Implements your Activity
Your activity is actually an observable. And it holds a list of observer.
```java
public class LifeWatcherDemo01 extends LifecycleActivity {

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_tv_btn);

        Watcher01 watcher01 = new Watcher01();
        getLifecycle().addObserver(watcher01);
    }

    public void onClickSimpleButton(View v) {
        startActivity(new Intent(this, B.class));
    }

    public void onClickSimpleButton2(View v) {
    }
}
```

**Benefit** : It decoupled your logic and your activity, which is great~
<br/>
**Cost** : LifecycleActivity is a subclass of FragmentActivity, which means this library does not support AppCompatActivity for now. And it is in alpha stage right now(2017.08.04), so it may not stable. If you use it, you may need to change if the library changes
And this library does not support Fragment, neither.


## 03. AOP
AOP is a great approach. If you are familiar with AspectJ, you would know what I am saying, and could know what to do immediately after you see the title. 

If you are not, that's okay. Here is a code snippet of how to do it
```java
@Pointcut("execution(* android.app.Activity+.on*(..)) && this(activity) ")
public void hijack(Activity activity) {
    ...
}
```

**Benefit** : it is very easy to write, you would love the feeling when you write AOP code. 
And it’s also super powerful. You can support the lifecycle of Fragment, Activity, View,… and whatever you want to know. 
<br/>
**Cost** :
But AOP may be a huge step forward if your project currently does not depende on AspectJ.  

p.s. here is how to hijack the lifecycle of Fragments
```java
@Pointcut("execution(* android.support.v4.app.Fragment+.on*(..)) && this(fragment) ")
public void hijack(Fragment fragment) {
    ...
}
```

## Bonus: How to make lifecycle component library support AppCompatActivity?
Actually this is not hard to do. In opposite, it is quite easy.

Here is what I do:
```java
public class LifeAppCompatActivity extends AppCompatActivity implements LifecycleRegistryOwner {
    private final LifecycleRegistry mRegistry = new LifecycleRegistry(this);

    @Override
    public LifecycleRegistry getLifecycle() {
        return mRegistry;
    }
}
```

And just make your activity derive from this new LifeAppCompatActiivty:
```java
public void MapActivity extends LifeAppCompatActivity {
    ...
}
```


## Conclusion
AOP is a little heavier than usual. If your project already uses AOP, you can try it. It's really really wonderful to write such decoupled code using AOP. Otherwise, consider Application.ActivityLifecycleCallbacks or LifecycleActivity.

If this is a generics logic that is good for every activity you have, the Application.ActivityLifecycleCallbacks should be the best solution for you.

If you are just want to inject some particular activities, LifecycleActivity should be good. Although it is still in alpha stage (now is 2017.08.03), so this library does not support Fragment and AppCompatActivity for now. 