[Hugo](https://github.com/JakeWharton/hugo) is a library made by Jake Wharton. I have studied the Hugo library in June 2016. But two years later, Junly 2018, I investigated the library again, and found out it could be improved. This post is about Hugo and how to improve it.

## I. Hugo

### 1.  What 
Hugo is a library that helps you log the method call and the time it took to execute.  So it would be convenient for the developers to know which method takes more time to execute. 

Simply add @DebugLog to your methods and you will automatically get all of the things listed above logged for free.
```java
@DebugLog
public String getName(String first, String last) {
  SystemClock.sleep(15); // Don't ever really do this!
  return first + " " + last;
}
```

``` java
V/Example: ⇢ getName(first="Jake", last="Wharton")
V/Example: ⇠ getName [16ms] = "Jake Wharton"
```

### 2. How
How does Hugo implement this functionality? 

### 2-1). Outline
Base what we saw before, Hugo uses an annotation, so Hugo must use APT(Annotation-Processing Tool) to know which method is needed to track.

Also, this log sort of thing seamlessly record a log, which seems like a AOP(Aspect-Orientated Programming). 

### 2-2). Annotation Processing
We normally use APT to generate a file to help us do something. Butterknife is a project that use APT to generate a file to findViewById and to register onClickListener. 

If you are not familiar with APT, I wrote three posts to introduce how to implement your own Butterknife. 
[Make Your ButterKnife 1/3](https://github.com/songzhw/songzhw.github.io/blob/master/open_src/2016-05-26-make-your-own-butterknife-1.md)
[Make Your ButterKnife 2/3](https://github.com/songzhw/songzhw.github.io/blob/master/open_src/2016-05-27-make-your-own-butterknife-2.md)
[Make Your ButterKnife 3/3](https://github.com/songzhw/songzhw.github.io/blob/master/open_src/2016-05-29-make-your-own-butterknife-3.md)
Throught these posts, you would get to understand how the APT works

However, I guess it wrong. Hugo is not using the APT to get the annotation information. Instead, Hugo is using **Gradle Plugin** to get a chance to inject the AspectJ code.

### 2-3). AspectJ
What Hugo does is writing a Gradle Plugin, in which it inject the AspectJ code to every annotated method. So we need to tell a little more about AspectJ. 

First of all, let's clarify some concepts
* **AOP** : Aspect-Oriented Programming.   You can simply think of AOP as a chance to inject some code to your code. You can do it before a method, after a method, or around a method. The typical usage is adding log before one method.
* **AspectJ** : It's a framework of AOP. Although it is somehow hard to be intergrated to Android projects.

Let's see an example, and you will get to know AOP.

```java
// You have a couple of Activity classes
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }
}


// And then you define a AOP class to inject something
@Aspect
public class AspectTest {
    private static final String TAG = "szw";
    @Before("execution(* android.app.Activity.on**(..))")
    public void onActivityMethodBefore(JoinPoint joinPoint) throws Throwable {
        String key = joinPoint.getSignature().toString();  // the method name
        Log.d(TAG, "onActivityMethodBefore: " + key);
    }
}

```

The code before does the injector job. 
1. @Before: means this onActivityBefore() method will get called when the condition is meet
2. "andorid.app.Activity.on**(..) : means every Activity's lifecycle method is the condition that trigger this onActivityBefore() method.
3. What we did above is to add a log to every activity lifecycle method.

Now you must have known the solution of Hugo. It is just doing the same. It insert a log to record the start time before one method, and also insert a log to record the end time after that method, so it would know the execution time of that method.  


## II. Gradle Plugin



## III. Improvement