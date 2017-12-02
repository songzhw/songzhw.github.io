## 1. Introduction
ViewModel 1.0.0 stable version is released at the begining of November. I happened to make a page that support the screen rotation, so I dug it a littler deeper.

At first, I found it is amazing to tell the difference between a normal destory of one activity and that kind of
destory due to configuration change. . And LiveData could have a observer? Hello, this is a typical MVVM architecture. I can see myself happily coding MVVM in the world of ViewModel and LiveData.

But when I dug a little deeper, I found out a couple of issues that makes me uncomfortable.

## 2. ViewModel: One thing you may need to pay attention
The reason of using ViewModel is that it can save and restore data through configuration change (i.e. screen rotation, system language change). I am not satisfied with that. So I was wondering, could the data in a ViewModel class survive the kill of one app?

Let's get our hands dirty and code a bit. The following code save a custom class(`User`) in the static and also in a ViewModel.

```java
public class ZeroViewModel extends ViewModel {
    public User user;
}
```

And the Activity.onCreate() is trying to get the data

```java
   vm = ViewModelProviders.of(this).get(ZeroViewModel.class);
   System.out.println("szw vm.user = " + vm.user);
```

Then I rotated the screen, the data`ZeroViewModel` are as same as the original one.
Finally, I pressed home to bring the app to the stack, then terminated the app through Android Studio, then brought the app back. What happened?
The user in the `ZeroVm` is `null` !!!

So now you should understand my point. Data in the ViewModel could survive the configuration change. But you still have to override onSaveInstanceState() to save data in case the app is killed when it is in the background.

This pictures shows how to ternimate an app in Android Studio. (First select the process, then click the "terminate application" button.)

![](./_image/2017-11-30-20-38-08.jpg)

  
So if you are trying ViewModel, jsut remember that you can not rely on ViewModel to save the data for you when the memory is low.
 
## 3. LiveData: too weak to use?
 
### 3.1 too many observers
The first impression about LiveData is that it help us decouple the logic and UI, so you could have a LiveData in the ViewModel class, and change the UI in the Activity class. This is beautiful, but development in real-world is much complex than a sample project.  You may need to add more than ten (or even more) observers to update the UI. This is kind of annoying and there would be a bunch of boilerplate in your code. We want to free the developer from boilerplate and boring development, so LiveData is a satisfying framework.
 
### 3.2 bugs of LiveData?
Even worse was the insufficient functionality of LiveData. LiveData may support saving data and notifying a change, but it does not work well with Event. By “Event”, I mean the “openDetailPageEvent”, “addNewItemEvent”, and so on. If you really want to save an Event in the LiveData, then you might find out you will get the notification immediately you rotate the screen. This is odd. You didn’t click the “add new item button”, how come did the LiveData send you a notification that the data is changed?
 
This is the issue of LiveData. It cannot work well with Event. If you want to send a Event, you could use the [SingleLiveEvent]( https://github.com/googlesamples/android-architecture/blob/dev-todo-mvvm-live/todoapp/app/src/main/java/com/example/android/architecture/blueprints/todoapp/SingleLiveEvent.java).
 
 
### 3.3 you should use EventBus for Events
SingleLiveEvent has its own limit. It can only have one observer, which may be against your requirement. Instead, I think we should not use SingleLiveEvent at all. When we need to send a event, the good solution for us should be [EventBus]( https://github.com/greenrobot/EventBus).   I know some people are not happy with the lose coupling in the project that used EventBus. But I prefer EventBus over the SingleLiveEvent in this scenario.
 
### 3.4 another strike
 
[todo-app]( https://github.com/googlesamples/android-architecture) is a sample app that created by Google. Google shows many clean and elegant architecture in this repo.   And [todo-mvvm-live]( https://github.com/googlesamples/android-architecture/tree/todo-mvvm-live) is a branch of this repo.  it uses some Architecture Components like ViewModel, LiveData, and other lifecycle-aware classes.  Ironically, even in Google’s own project, they are not using LiveData to store the data.  For the boilerplate issue, Google was using [data-binding]( https://developer.android.com/topic/libraries/data-binding/index.html#data_objects) to update the UI automatically once the data is changed. So you don’t have to write so many observers to update UI.
 
And for the event, this todo-mvvm-live project is using the SingleLiveEvent I’ve mentioned, which is not a good idea in my opinion. Now the code is quite weird, and you can only have one observer for each SingleLiveEvent. Instead, you should use EventBus to send and receive events.
 
## 4. Conclusion
I like the idea of architecture component library. It helps us decouple the fat Activity class, especially the lifecycle library.
But ViewModel are not that good as I thought it would be. And LiveData will result a bunch of boilerplate and even bugs. I am not a fan of ViewModel and LiveData.
However, the ViewModel and LiveData is only in their 1.0.0 version (today is Nov 30, 2017). Who knows what they will grow into? So let’s keep eyes on them and maybe they will surprise use in the future.

 

