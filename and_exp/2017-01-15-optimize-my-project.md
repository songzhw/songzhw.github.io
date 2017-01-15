In the last four months, I did some optimization to our company's project.

####. Method Count
Some code are written like this:

```java
public void onCreate(Bundle b){
    initData();
    initView();
}
```

The code separate the initilization into two methods in every Activity's onCreate() method. My question is : "Does it necessary?"

Normally, it's fine. But things are different on Android. Android has a 65536 method count limit. Once you pass that limit, thanks to Google, you can still build your project using Google's multidex, but your app is now in a more complex zone. 

Sometimes, you may find some "ClassNotFound" crash because of multidex.

And multidex will make your start up time very long. Because Android need to unzip the multiple dexes and run the app. And this takes time. I saw some project even have a ANR when starting up. That's because of multidex. Once they remove a lot unused method and business logic, this ANR disappear. 

I'm not saying multidex is evi. I just want to get your attention and tell you multidex has some major disadvantage. So if you can, please try your best to shrink you method count below 65536. Proguard will be a helper to help you achieve that. 

Now back to our example, these init***() methods are actually not necessary. You can just avoid this by using comments. Comments are very clear too. You do not need multiple methods to show your intention.

```java
public void onCreate(Bundle b) {
    // init data
    ...
    // init view
    ...
}
```

####. Enum

 


1. emnu

2. method count

3. separate logic from UI

4. add unit test

5. open-close principle

