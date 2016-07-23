## static field

Static field has a long lifecycle that outlive all the activities. Its memory will still exist until you exit your application. In this case, if a static variable holds a reference to one Activity, the Activity's memory actually leaks. 

### Example
One example most people would never thought of is the **Singleton**. You may wonder, "What? Singleton that we used a lot will cause memory leak? That's insane." Yes, the singleton may cause memory leak if you do not pay attention to it. 

A typical Singleton code is as followed:

```java
public class Singleton{
    private static Singleton instance;
    private Singleton(){}
    public static Singleton getInstance(){
        if(instance == null){
            instance = new Singleton();
        }
        return instance;
    }
}
```

How about this? If the singleton needs to do something using a Context, and you gives it your Activity. Yes, the memory leaks.

The followed code is a bad example, and it will cause the leaks:

```java
public class Singleton2{
    private static Singleton2 instance;
    private Singleton2(Context ctx){
        // ... do somehting, hold the ctx object
    }
    public static Singleton2 getInstance(Context ctx){
        if(instance == null){
            instance = new Singleton2(ctx);
        }
        return instance;
    }
}
```

### Conclusion
You need to pay more attention to the static variable since the static field will exist long time in your application, especially to the static variables which holds the reference to Activity.