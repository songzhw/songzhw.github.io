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
