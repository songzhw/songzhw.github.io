
## Write Unit Test in Kotlin

Latest Android Studio(3.0 Canary 7) allow us to use Kotlin to write source code and test case too.  I’ve written a post about how to test kotlin code in java unit test files. Today, I want to talk about how to test kotlin code in kotlin. And it is not as that easy as you might think.

### 1. Mock
We’ve written the following java test code many many times:
```java
public FooTest {
    @Mock private IView view;
    
    @Before public void setUp(){
        MockictoAnnotation.init(this);
    }
}
```

But this does not work in kotlin. Kotlin does not allow you declare a field without giving it a value. 
In this case, view filed should be assigned a value, which is impossible for us. We can not initialize it until the setUp() method is invoked. 

So the `lateinit` value is here to rescue us
```java
public FooTest {
    @Mock lateinit var view: IView
    
    @Before public void setUp(){
        MockictoAnnotation.init(this);
    }
}

```

### 2. init again
Now we want to test a presenter. Still we can not just use `private Presenter presenter` in kotlin.  Besides, the initialization of presenter depends on the view field in the code above. 

Here is what we do
```kotlin
public FooTest {
    @Mock lateinit var view: IView
    var presenter : Presenter by lazy {
        Presenter(view)
    }
    
    @Before public void setUp(){
        MockictoAnnotation.init(this);
    }
}
```

### 3. static import
Kotlin does not support features like “static import”, which is used very frequently in the test case. You must see a lot of `onView().perform().check()`, or `assertEquals()`, or `any(***Class)`, or `mock(***Class) and so on.  

Now since we cannot use static import, we have to use the boring `Mockito.mock()`

But not be frustrated, we still have a way to achieve the static import effect by do this:
```kotlin
import org.mockito.Mockito.*
…
val ids = longArrayOf(9517717L)
verify(view).jumpToDetilsPage(9517717, ids)
```

### 4. Robolectric
Some test cases need Robolectric to test some special class. But since the `Class` object is different between Java and Kotlin. 

We may have a little confused about how to import Robolectric. Just like the screenshot below:
![](./_image/2017-07-19-21-26-51.jpg)



Here is what we do:
```kotlin
@RunWith(RobolectricTestRunner::class)
@Config(constants = BuildConfig::class, sdk = intArrayOf(Build.VERSION_CODES.LOLLIPOP))
class PresenterTest {
```
