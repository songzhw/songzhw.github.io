##Introduction
Consider, for example, the requirement that asked you to register location SDK on activity.onResume(), and, of course, unregister it on activity.onPause(). You may just add these two register and unregister code to every Activty that need to detect the location change. But this is ugly, and there are duplicate code.

Injecting the activity's lifecycle is a good solution. So you don't have to add the same logic in every activity that needs to know the change of location. 

But how could we inject the activity's lifecycle. Normally there are three ways to do that. In this post I will introduce three ways to implement this.

## 01. Application.ActivityLifecycleCallbacks


## 02. LifecycleActivity


## 03. AOP

## Conclusion
AOP is a little heavier than usual. If your project already uses AOP, you can try it. It's really really wonderful to write such decoupled code using AOP. Otherwise, consider Application.ActivityLifecycleCallbacks or LifecycleActivity.

If this is a generics logic that is good for every activity you have, the Application.ActivityLifecycleCallbacks should be the best solution for you.

If you are just want to inject some particular activities, LifecycleActivity should be good. Although it is still in alpha stage (now is 2017.08.03), so this library does not support Fragment and AppCompatActivity for now. 