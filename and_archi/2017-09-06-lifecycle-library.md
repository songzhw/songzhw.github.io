Google has released the Architecture Component library since Google I/O 2017. Now it's nearly half year since then. I think it's time to talk about it right now. Today I will focus on the lifecycle library.

### Why I Think Fragment Is a Failure
Getting back to Year 2011, Google released its Android 3.0. The biggest change of Android 3.0 is that it supports tablet. The second biggest change of it is the Fragment.

The official concept of Fragment is "A fragment represents a behavior or a portion of user interface in an Activity." And this also points out what is bad about Fragment. It could a behavior, or it could be a part of UI. 

A class does more than one responsibility may the source of your problems.  A typical example of such a class is `Activity`. Activity could deal with touch event, it could deal with all the UI (findViewById, request full screen UI, ...), and it also could send Http request.  Normally our solution is MVP. MVP could decouple your code, separate the logic and UI, so your code could be testable, maintable, stable. 

Like I said , Activity has a tendency to couple logic and UI. The invention of Fragment did help Activity with it. Fragments only makes the problem smaller , as your Activity may contain a couple of Fragment, do not make this problem vanish. 

Of course, Fragemnt has other disadvantages. But for our architecture, Fragment can not help us improve our app. That's why I said Fragment is a failure.

### The Beauty of the Lifecycle Library
Fourtunately, Google finally realized the tendency of couple logic and UI in the Activity, and release the Architect Component library. 

The Architect Component library actually is a collection of libraries: Lifecycle, Room, LiveData, and ViewModel. The lifecyle library is the brightest library in my eyes, and it exactly solve the coupling problem I just mentioned. 

Let's take a old code as an example. 
```java
public class FeatureListActivity extends Activity{
    @Override
    public void onResume(){
        // for featureA
        gcm.register(this);
        
        // for featureB
        FeatureBUtils.addObserver(this);
        
        // for featureC
        featureC.setContext(this);
        featureC.refreshAd();        
    }
    
    @Override
    public void onPause(){
        gcm.unregister(this);
        FeatureBUtils.removeObserver(this);
        featureC.setContext(null);
    }
}
```

Oh, this FeatureListActivity may contains a couple of new features, but the above code is quie ugly and coupling. Let's say now you want to modify some behavior in Feature C, now you need to find out all the code that related to Feature C in the Activity, and modify one or two of them very carefully.  

This is not good for the code, for the future. But how could we improve this?
Bingo! The lifecycle library. 


### How to Use the Lifecycle Library?
Let's see how we could improve the above example.

1. add the architect component library

2. add a observer 

3. modify the existing Activity


### Question1: How About AppCompatActivity?


### Question2: How to deal with onRequestPermissionResult()?


### Question3: How is it compatible with MVP?



