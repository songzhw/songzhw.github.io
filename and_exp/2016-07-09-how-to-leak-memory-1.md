#How to leak memory on Android?
Every time we talk about memory is how to detect a memory leak, and how to fix it. This time, let's think it an opposite way: how to leak memory on Android?

This is a series of maybe three posts. Today I will introduce you the first post.

## Leak your memory : way 1
Once you claim to have some resources, you use it, and you do not return it. That way, the resource is still open when you finish one Activity, that will cause our Activity to be holded  by the resource, and the GC cannot release the reference to Activity.

For example, BroadcastReceiver. Once you register a BroadcastReceiver in the onCreate() function of a Activity, and you do not call unregister method in the onDestory() method, you may cause a leak when you exit the Activity.

Typical situations are as followed:
* BroadcastReceiver
* Cursor, In/OutputStream
* Listener (resgister, unregister)

### Another Example: ContentObserver
ContentObserser is a class in the "android.database" package. It receives call backs for changes to content. If a content change occurs, the onChanged() method will get called.

A more detailed example. You call the code below in the onCreate(). And you do not clear it in the onDestory(). Of course, you will get a leak.

```java
  private ContentObserver observer = new ContentObserver(mHandler) {
        @Override
        public void onChange(boolean selfChange) {
            super.onChange(selfChange);
            // ... do something
        }
    };
    
...

activity.getContentResolver()
    .registerContentObserver(...., observer);
```
Then how to fix this?
The answer is simple. Just unregister is okay.

```java
activity.getContentResolver()
    .unregisterContentObserver(observer);
```
## Conclusion
In conclusion, if you register your activity to some other class, remember you have an opportunity to exit from there and you do use the opportunity when you exit the page. 










