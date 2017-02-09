
### Requirement
We have a requirement that show the in-app notification when our app is forground, and show the Android system notification when our app is background or not started. 

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


#### Plan C : ActivityLifeCycleCallbacks 

















