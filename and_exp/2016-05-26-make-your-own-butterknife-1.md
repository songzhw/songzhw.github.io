# Make your own ButtferKnife
ButterKnife is a famous library developed by Jake Wharton. This is a sharp knife that can help you reduce a lot of boilerplate. 

Here is how to use it:

``` java
class ExampleActivity extends Activity {
  @BindView(R.id.user) EditText username;
  @BindView(R.id.pass) EditText password;
  
  @OnClick(R.id.submit) void submit() {  }

  @Override public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.simple_activity);
    ButterKnife.bind(this);
    // TODO Use fields...
  }
}
```

Today , instead of using it, I want to show you how to make such a knife. It contains more details than you first think. So I may need to two or three posts to explaint it clearly. 

# v0.1 : Runtime ButterKnife

## 1. intruductin of runtim annotation
Firstly, you need to be familiar with Java's annotation.  Annotation is a kind of comment or meta data you can insert in your Java code. These annotation can then be processed at compile time by pre-compiler tools, or at runtime via Java Reflection. 

In short, java annotation is a tool that can help you add extra info in your code.   If you are not familiar with annotation, there are tons of articles in the Internet. Please go check them.

Secondly. Annotation can be divided into three types : runtime annotain, compile-time annotaion, source-level annotation. 

Runtime annotation can be read by java Reflection in the run time. In this way, I can read the extra infomatin hiding in the annotation in the runtime.

## 2. write the code

#### 2-1. Annotation
Firstly, I create an annotation:
```java
@Retention(RetentionPolicy.RUNTIME) 
@Target(ElementType.FIELD )
public @interface InjectView {
    int value();
}
```

Note that I used the `RetentionPolicy.RUNTIME` to declare that this @InjectView annotation is a runtime annotation.  

And this @InjectView annotation is used to declare the filed (`@Target(ElementType.FILED)`).

the `value()` method is our container for the extra information. In this case, it is the id of some specific view. 

#### 2-2. Use this annotation in Activity
```java
public class SimpleActivity extends Activity {
    @InjectView(R.id.title) 
    TextView tvTitle;

    @Override 
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.simple_activity);

        Views.inject(this);
        // ... ...
    }
}
```


In this way, you can avoid the boilerplate like :
```java
tvTitle = (TextView) findViewById(R.id.title);
```
But, how does it work? the `2-3` section will reveal the   working principle. 

#### 2-3. How does this annotation work?
The trick is actually in the `Views.inject()` method. Let's have a look at this method.

```java
public class Views{
    public static void inject(Activity activity) {
        Class<?> clz = activity.getClass();
        Field[] fields = clz.getDeclaredFields();
        for (Field field : fields) { // find every field declared by @InjectView
            if (field.isAnnotationPresent(InjectView.class)) {
                InjectView injectView = field.getAnnotation(InjectView.class);
                int id = injectView.value(); // get the extra info
                field.setAccessible(true);
                field.set(activity, activity.findViewById(id)); 
            }
        }
}
```
What the Views class does is iterating all the fields of our Activity, then finding all the field declared by @InjectionView. Then it extract the value (the id of the view) out and call the `findViewById()` method automatically. 



## 3. disadvantage of runtime annotations
Now, our version 0.1 is done. However, is is good enough? I'm afraid not.

We all know that reflection is a little expensive in Android. If we can remove the reflection, aka, remove the runtime annotation, the performance will be better. And that is what we will talk about in my next post.
