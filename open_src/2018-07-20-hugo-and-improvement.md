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
We solved how to record time for every annotated method, but I also said that integrating AspectJ is quite complex for any project. So to help you make it easy, Hugo wrote a gradle plugin to integrate AspectJ for you. Then all you have to do is just to add `apply plugin: 'hugo'`, nothing more. You don't need to add a couple of dependencies, and not need to  configure lots of options.

This is the structure of gradle plugin module. And I will tell you how to create such a module.

![](./_image/2018-07-24-10-22-38.jpg)

### step 1. create a java module 
First of all, create a java module. And then you will get a build.gradle generated. This build.gradle is not good enough for us, we need to add something more.

Gradle is writen by Groovy. And Groovy is such a powerful language that also makes our life easier, so we need to import groovy in this module.
`apply plugin: "groovy"`

Since this is a gradle project, we need to add gradle api here. Otherwise, javac doesn't know what a `Plugin` class is.

```groovy
dependencies {
  compile gradleApi()
  compile localGroovy()
}
```

To write some AOP code, we need to import AspectJ too. So we add more dependencies.

```groovy
dependencies {
  compile gradleApi()
  compile localGroovy()
  compile 'com.android.tools.build:gradle:1.3.1'
  compile 'org.aspectj:aspectjtools:1.8.6'
  compile 'org.aspectj:aspectjrt:1.8.6'
}
```
### step 2. create a plugin
Gradle plugin is kind of like a combination of dependencies + extension + task. What you need Gradle to do is writen in one plugin.

So we need to create a class called `HugoPlugin.groovy`. Here is something you need to pay attention. We need to add the configuration to tell gradle which plugin we are using as well. This could be done by add a *.properties file to `resources/META-INF/gradle-plugins` folder .

![](./_image/2018-07-24-10-30-41.jpg)

You just need to list all the plugins in the properties file. Take hugo.properties for example, its content is :

```properties
implementation-class=hugo.weaving.plugin.HugoPlugin
```

Also, the name of this properties file (here is "hugo") is meaningful too. If your properties file is "a.properties", then you could use `apply plugin: "a"` in your app module. 

### step3. complete the plugin     
Every gradle plugin needs to implement one method. 

```groovy
class HugoPlugin implements Plugin<Project> {
    @Override void apply(Project project) {
            ... ...
    }
}
```

What hugo did is add the dependencies of AspectJ, and bind this AOP behavior to the javac.

1). add AspectJ

```groovy
  project.dependencies {
      debugCompile 'com.jakewharton.hugo:hugo-runtime:1.2.2-SNAPSHOT'
      debugCompile 'org.aspectj:aspectjrt:1.8.6'
      compile 'com.jakewharton.hugo:hugo-annotations:1.2.2-SNAPSHOT'
    }
```

2). bind it to javac
```groovy
      JavaCompile javaCompile = variant.javaCompile
      javaCompile.doLast {
        String[] args = [
            "-showWeaveInfo",
            "-1.5",
            "-inpath", javaCompile.destinationDir.toString(),
            "-aspectpath", javaCompile.classpath.asPath,
            "-d", javaCompile.destinationDir.toString(),
            "-classpath", javaCompile.classpath.asPath,
            "-bootclasspath", project.android.bootClasspath.join(File.pathSeparator)
        ]
```

3). You may have the question, that where is the AOP code?
Good question. That's because all the AOP code are inside the hugo-runtime library. Once you add the hugo-runtime library, then AOP code is already in your code. So no worries.

```groovy
project.dependencies {
      debugCompile 'com.jakewharton.hugo:hugo-runtime:1.2.2-SNAPSHOT'  //===> This is the AOP code
```

## III. Improvement








