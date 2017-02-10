
### Requirement
We have a requirement that show the in-app notification when our app is foreground, and show the Android system notification when our app is background or not started. 

### Challenge 01 : UI
The challenge is you have to show a floating window on top of the top Activity, and many Activities are from third party library. That's to say you have to show a floating windown even on top of some activities which you do not have source code of. 

Okay, That's not that difficult if you understand how to make a floating window. 

I have more than three solution to create a floating window:
* Fragment/Dialog : not that great. Because we are not sure every Activity support Fragment. Activity below Android 3.0 (not a FragmentActivity) does not support Fragment. And Dialog will show a semi-transparent background, which we do not need in this case.
* WindowManager : This is doable. However, it may have many problems that need you to solve. I will name some of these problems for you, so you can understand how complex it is:
    * TYPE_PHONE is forbidden by default on Android 6. You have to jump out of your app to let user permit show the floating window.
    * Floating window with TYPE_PHONE type, will show on top of every app. This is not our requriement asked. Our requirement asked me to show the floating notification only on top of our own app.
    * Floating window with TYPE_TOAST does not get the touch event below Android 4.4. And in some cases, the floating window may disappear itself after some time.
    * ...
* DecorView : a good solution, and good for our requirement. It only shows on top of our app, and it will always be there in our app until you remove it.

By the way, if you are not familiar with DecorView, please see [the post I wrote one years ago](https://github.com/songzhw/songzhw.github.io/blob/master/ui/2016-08-23-action-sheet.md). 

The conclusion is :
* If you want to show your floating window through all apps, you should choose `WindowManager`
* If you just want to show a floating window in your specific Activity, you should choose `DecorView`

### Challenge 02 : Current Activity

#### Plan A : `List<RunningTaskInfo>`
The first method I thought of is the `RunningTaskInfo` list. Just like this:

```java
public static boolean isAppOnForeground(Context context) {
    try {
        int maxRunning = 100;
        ActivityManager am = (ActivityManager) context.getSystemService(ACTIVITY_SERVICE);
        List<ActivityManager.RunningTaskInfo> taskList = am.getRunningTasks(maxRunning);
        if (taskList == null || taskList.isEmpty()) {
            return false;
        }
        ActivityManager.RunningTaskInfo info = taskList.get(0);
        String name = info.topActivity.getPackageName();
        if (context.getPackageName().equals(name)) {
            return true;
        }
    } catch (Exception e) {
        e.printStackTrace();
        return true;
    }
    return false;
}
```

However, what we need is apparently the reference of the top Activity. So the previous code does not work because it only get the Activity name, not the reference.

#### Plan B : Ordered Broadcast
My colleague thought of a solution. In this action, your Activity will register a broadcast receiver that will show the in-app notification, and your application needs to register another broadcast receiver to show to system notification.

When the GCM push is here, what you need to do is just send a ordered broadcast. If your Activity is existing (ActivityLifeCycleCallback will tell you that), your Activity's receiver will consumer the push message and abort the ordered broadcast. If not, then no one will abort this ordered broadcast, then your another broast will get the broadcast and show the system notification.

A good plan. But it is a little complexer than I thought. I have to register and unregister for every top Activity, and define at least two type receivers. So I moved to another solution.

#### Plan C : ActivityLifeCycleCallbacks 
Plan B just said, we can use ActivityLifeCycleCallback to know if your Activity is showing foreground. 

##### C-01 : Introduction of ActivityLifeCycleCallbacks
Here is how it did that:

```java
public class AndApp extends Application {

    @Override
    public void onCreate() {
        super.onCreate();
        this.registerActivityLifecycleCallbacks(callbacks);
    }

    @Override
    public void onTerminate() {
        super.onTerminate();
        this.unregisterActivityLifecycleCallbacks(callbacks);
    }

    // ========================= CurrentActivity =========================
    private ActivityLifecycleCallbacks callbacks = new ActivityLifecycleCallbacks() {
        @Override
        public void onActivityStarted(Activity activity) {
           // ▼ record this "activity" is what you need, the Activity foreground
        }
    };

}
```

Note that the ActivityifecycleCallbacks will take affect in every Activity in your app. Yes, even those activities that are in the 3rd party library. 

##### C-02 : Record the Foreground Activity
What we need is just a field, a static or a singleton will be both okay for us, to record the foreground Activity. 

So I created a helper class to do it:

```java
public class ForegroundActivityHelper {
    private static ForegroundActivityHelper instance = new ForegroundActivityHelper();
    private WeakReference<Activity> foregroundActivityWeakRef;


    private ForegroundActivityHelper() {
        // singleton. This constructor is empty.
    }

    public static ForegroundActivityHelper getInstance() {
        return instance;
    }

    public Activity getForegroundActivity() {
        Activity currentActivity = null;
        if(foregroundActivityWeakRef != null){
            currentActivity = foregroundActivityWeakRef.get();
        }
        return currentActivity;
    }

    public void setCurrentActivity(Activity activity){
        foregroundActivityWeakRef = new WeakReference<Activity>(activity);
    }

}
```


When we get the GCM push, we will dispatch the push message like this:

```java
public void dispatchPush(String msg){
    Activity actv = ForegroundActivityHelper.getInstance().getForegroundActivity();
    if(actv != null){
        // ▼ show the in-app notification
    } else {
        // ▼ show the Android system notification
    }
}
```

Here is our real code of ActivityLifecycleCallbacks:

```java
        @Override
        public void onActivityStarted(Activity activity) {
            ForegroundActivityHelper.getInstance().setCurrentActivity(activity);
        }

        @Override
        public void onActivityStopped(Activity activity) {
            ForegroundActivityHelper.getInstance().setCurrentActivity(null);
        }
```


Easy, right? But when we really do it, we got a bug. 

##### C-03 : A bug: Why? How to fix it? 
When you open one Activity from another Activity , and then you get a GCM push. Our expectation is showing a in-app notification. After all, our activity is still foreground. But no, your app show a system notification instead. Why?

Put some logs in your app, then we find out why. Here is the log, when we open a DemoActivity from HomeActivity:
```
02-07 11:32:30.396 : szw start() ca.six.notify.HomeActivity@f7159fc

02-07 11:32:32.725 : szw start() ca.six.notify.decor.DemoActivity@1d9cd18 
02-07 11:32:34.600 : szw stop()  ca.six.notify.HomeActivity@f7159fc  

02-07 11:32:35.398 : szw consumer : msg = GCM Message : 1486485155398  
02-07 11:32:35.398 : szw call the system notification

```

From this log, we can clearly see why we got a wrong notificaiton. Because the DemoActivity.onStart() got called first, and the HomeActivity.onStop() got called later. That's why our ForegroundActivityHelper recored a null reference. 

Then how do we fix it? Don't worry, what you need to do is just keep a flag to the `ForegroundActivityHelper`. Normally, if the app is still in the foreground, we will never call `setForegroundActivity(null)`. When the app is in the background, then we will call `setForegroundActivity(null)`.

So the solution is like this:

```java

public class AndApp extends Application {
    private int activityRefCount = 0;  // ▼ This is the flag we need.

    @Override
    public void onCreate() {
        super.onCreate();  
        this.registerActivityLifecycleCallbacks(callbacks);
    }

    @Override
    public void onTerminate() {
        super.onTerminate();
        this.unregisterActivityLifecycleCallbacks(callbacks);
    }

    // ========================= CurrentActivity =========================

    private ActivityLifecycleCallbacks callbacks = new ActivityLifecycleCallbacks() {
        @Override
        public void onActivityStarted(Activity activity) {
            activityRefCount++;
            ForegroundActivityHelper.getInstance().setCurrentActivity(activity);
        }

        @Override
        public void onActivityStopped(Activity activity) {
            activityRefCount--;
            if(activityRefCount == 0) {
                ForegroundActivityHelper.getInstance().setCurrentActivity(null);
            }
        }
    };

}
```


### Conclusion
By creating a floating notification, we learned how to :
* create a floating window by different ways
* how to inject all activities's lifecycle
* how to get a referenece of your forground (if in background, then get a null)




