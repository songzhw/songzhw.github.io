## 1. Introduction
ViewModel 1.0.0 stable version is released at the begining of November. I happened to make a page that support the screen rotation, so I dug it a littler deeper.

At first, I found it is amazing to tell the difference between a normal destory of one activity and that kind of
destory due to configuration change. . And LiveData could have a observer? Hello, this is a typical MVVM architecture. I can see myself happily coding MVVM in the world of ViewModel and LiveData.

But when I dug a little deeper, I found out a couple of issues that makes me uncomfortable.

## 2. ViewModel: Couldn't save data when necessary
The reason of using ViewModel is that it can save and restore data in a configuration change (i.e. screen rotation, system language change). I am not satisfied with that. So I was wondering, could the data in a ViewModel class survive the kill of one app?

Let's get our hands dirty and code a bit. The following code save a custom class(`User`) in the static and also in a ViewModel.

```java
public class SameVm {
    public static User user;
}

public class ZeroViewModel extends ViewModel {
    public User user;
}
```

And the Activity.onCreate() is trying to get the data

```java
   vm = ViewModelProviders.of(this).get(ZeroViewModel.class);
   System.out.println("szw vm.user = " + vm.user);
```

Then I rotated the screen, the data in `SameVm` and `ZeroViewModel` are as same as the original one.
Finally, I pressed home to bring the app to the stack, then terminated the app through Android Studio, then brought the app back. What happened?
The user in `SameVm` and `ZeroVm` are both `null` !!!

So now you should understand my point. I expected the data in a ViewModel could survive configuration change and app termination. But it does not.  It only survive the configuration change. 

This pictures shows how to ternimate an app in Android Studio. (First select the process, then click the "terminate application" button.)

![](./_image/2017-11-30-20-38-08.jpg)


For that, I really don't understand. It's the same data, which is in the ViewModel class. If you can save and restore it in a configuration change, why can't you just do the same thing to the app terminatation.
age you need to claim like more than twenty observer to update the UI when the data is changed. It’s kind of annoy and there are too many boilerplate. 

 
By the way, this is how you cannot replace onSaveInstanceState() with ViewModel. So for you, you still need to persist your data in the onSaveInstanceState() and restore it in the onCreate(). 
 
And by comparing the ViewModel and static class, I found out the difference between saving a data in a ViewModel and saving it as a static value is quite small. Of course, you can say, static value could be modified by many classes, and one ViewModel can be modified by its corresponding Activity. I agree that, but we can make a rule to say we have different static data class for different value to survive the configuration change.
 
## 3. LiveData: too weak to use?
 
### 3.1 too many observers
The first impression about LiveData is that it help us decouple the logic and UI, so you could have a LiveData in the ViewModel class, and change the UI in the Activity class. This is beautiful, but development in real-world is much complex than a sample project.  You may need to add more than ten (or even more) observers to update the UI. This is kind of annoying and there would be a bunch of boilerplate in your code. We want to free the developer from boilerplate and boring development, so LiveData is a satisfying framework.
 
### 3.2 bugs of LiveData?
Even worse was the insufficient functionality of LiveData. LiveData may support saving data and notifying a change, but it does not work well with Event. By “Event”, I mean the “openDetailPageEvent”, “addNewItemEvent”, and so on. If you really want to save an Event in the LiveData, then you might find out you will get the notification immediately you rotate the screen. This is odd. You didn’t click the “add new item button”, how come did the LiveData send you a notification that the data is changed?
 
This is the issue of LiveData. It cannot work well with Event. If you want to send a Event, you could use the [SingleLiveEvent]( https://github.com/googlesamples/android-architecture/blob/dev-todo-mvvm-live/todoapp/app/src/main/java/com/example/android/architecture/blueprints/todoapp/SingleLiveEvent.java).
 
 
### 3.3 you should use EventBus
SingleLiveEvent has its own limit. It can only have one observer, which may be against your requirement. Instead, I think we should not use SingleLiveEvent at all. When we need to send a event, the good solution for us should be [EventBus]( https://github.com/greenrobot/EventBus).   I know some people are not happy with the lose coupling in the project that used EventBus. But I prefer EventBus over the SingleLiveEvent in this scenario.
 
 

