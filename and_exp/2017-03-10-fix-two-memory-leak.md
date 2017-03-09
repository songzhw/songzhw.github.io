I searched the memory leak in our company's project and did find some leaks. Here is the two leaks I fixed.

### How to find memory leak?
Thanks to [Square company](https://github.com/square), we now have a memory leak detect library, called [LeakCanary](https://github.com/square/leakcanary). All you need to do is just follow the LeakCanary's ReadMe, add the dependency and add a few lines of code in your Application class. If you have leak, LeakCanary will tell you that.


### Leak 01 static Context
Our code has a Activity, and its code is as below:

```java
public class SomethingActivity extends Activity{
    static Context self;

        @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        self = this;
    }
}
```

This leak is very easy to spot. `self` is a static field. A static field has a long lifecycle that is as long as the whole application. So even you want to finish this SomethingActivity, Android could not GC it since this Activity object still have strong reference.


### Leak 02 : ClientSession
Our app has a strict security policy. For example, if the user does not perform any action for more than 1 minutes, our app will log out this user.  And the `ClientSession` is in charge of this functionality.

But I got a memory leak report about this class:


#### 2.1 why leak?
The code of `ClientSession` is as below:

```java

```


Just like the first leak, we know the reason is the Singleton, aka, static field, holds the reference of our Activity and never release it. So a leak happens. 

But how do we fix that? It's kind of tricky. 

#### 2.2 solution 01

The first solution I thought is remove this Singleton, and use a normal class. And in every activity's onDestory(), we just cancel this timer. 

But after I really applied this solution, I found out this is not correct. Because cancelling the timer in onDestory() is not correct. If we jumps from A screen to B screen, A's timer should stop. But since the cancelling timer is in the onDestory(), so A's timer does not stop. So this solution fails.


#### 2.3 solution 02
Then I moved the cancelling action to onStop(), instead of onDestory(). But I found out I was wrong again. Because the requirement asks us to keep ticking even the user press home button. 

If we cancel the timer in onStop(), and the user press home button, then the timer stopped, which is not correct.


#### 2.4 solution 03