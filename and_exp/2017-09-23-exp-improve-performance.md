Here are some experience of me trying to improve the performance of apps recently. Hope it will help you when you try to do such optimization too. 

### 1. Layout Performance
Last month, I got a bug to fix. After understand how the existing code works, I fixed it quickly. But when I was trying to figure out the old code, I found out something else that was troubling me. 

The image below is the thing that really made me uncomfortable. Really, this layout needs so many layout to wrap the real content?

![](./_image/2017-09-23-20-24-01.jpg)

As we can see, it has seven layout at the bottom. The last six layout seems has nothing to do but wrap another layout. That said, the last six layout seems like unnecessary. 

Android has a article about [perfermance and view hierarchies](https://developer.android.com/topic/performance/rendering/optimizing-view-hierarchies.html) that you should read. Here is what it said.
```
Android Layouts allow you to nest UI objects in the view hierarchy. This nesting can also impose a layout cost. 

Each nested layout object adds cost to the layout stage. The flatter your hierarchy, the less time that it takes for the layout stage to complete.
```

This article even gives you a specific example
```
If you are using the RelativeLayout class, you may be able to achieve the same effect, at lower cost, by using nested, unweighted LinearLayout views instead. 
```

All this layout wants to do is just to draw a simple layout. So I refactor it, I just used one RelativeLayout which contains multiple views, and fulfill the same UI. But my layout hierarchy is two levels, rather tahn the eight levels in the previous images. Therefore, the performance gets improved. 

### 2. Unreasonable Registration
Like most of aps, the app I am working on has a settings page. The first page of our app is the HomeActivity. We have a requirement that if the user turn on or turn off one setting, our HomeActivity should get notified, and updated the UI.

This is not hard, am I right? You could have a listener, or an observer (that depends on which you prefer to call it). When the setting is turn on/off, a singleton class notified all the observers and ask them to notify themselves. 

This is how our app implements this requirements.
```java
[HomeActivity.java]
public void onResume(){
    SomeFeatureSettings.addObserver(someFeatureObserver);
    refresh();
}

public void onPause(){
    SomeFeautreSettings.removeObserver(someFeatureObserver);
}

public Observer someFeatureObserver = new Observer(){
    public void update(Observable obj, Object arg){
        refresh();
    }
};
```
This works just like what our business man wants. But is this code perfect, or good for performance? I have to say no. 

The previous code works only because you call "refresh" every time you call "onResume()"!  Yes, you removed the observer way too early, so when the user turn on/off this feature in the setting page, the HomeActivity page will never get notified.  The way my app did works just because it called refresh() in the onResume() method, which means you will still call refresh() method even the user does not turn/off the setting of that feature. These calls is unnecessary and make the HomeActivity takes more time to be brought to the front of our user.

The correct way to register an observer is like this:
```java
public void onCreate(){
    SomeFeatureSettings.addObserver(someFeatureObserver);
}

public void onDestory(){
    SomeFeautreSettings.removeObserver(someFeatureObserver);
}

public Observer someFeatureObserver = new Observer(){
    public void update(Observable obj, Object arg){
        refresh();
    }
};
```



