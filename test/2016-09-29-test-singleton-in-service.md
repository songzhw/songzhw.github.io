Recently I had a big problem : How to test singleton in a Service? I finally managed to fix this problem. But I think the process of how to solve this is a great opportunity to explain unit test clearly. So I write this post. 

## Our Service

```java
// [PushService.java]
public class PushService extends Service {
    public void onMessageReceived(String id, Bundle data){
        FooManager.getInstance().receivedMsg(data);
    }
}
```

And our FooManager is an instance:
```java
// [FooManager.java]
public class FooManager {
    private static FooManager instance = new FooManager();

    private FooManager(){}

    public static FooManager getInstance(){
        return instance;
    }

    public void receivedMsg(Bundle data){
    }
}
```

So what should we test the PushServie?

Of coures, we want to make sure the FooManager do call receiveMsg(). So what we want is like this: 

```java
verify(fooManager).receiveMsg(data);
```

Anyone who knows Mockito knows you have to make `fooManager` as a mocked object when you call `verify(fooManager)`. Otherwise, you will get an exception : `org.mockito.exception.misusing.NotAMockException`

So we should focus on mocking an instance of FooManager. I break down the test into two small tests: 
1. mock a singleton
2. mock a singleton in a Service

## Mock Singleton(1)

### Step 01 : Use Mockito to mock FooManager (Failed)
Firstly, I write a test:

```java

public class FooManagerTest {
    @Test
    public void testSingleton(){
        FooManager mgr = Mockito.mock(FooManager.class);
        Mockito.when(FooManager.getInstance()).thenReturn(mgr);

        FooManager actual = FooManager.getInstance();
        assertEquals(mgr, actual);
    }
}

```

But running this test, I failed and got an exception:

```
org.mockito.exceptions.misusing.MissingMethodInvocationException:
when() requires an argument which has to be 'a method call on a mock'.
For example:
    when(mock.getArticles()).thenReturn(articles);
Also, this error might show up because:
1. you stub either of: final/private/equals()/hashCode() methods.
   Those methods *cannot* be stubbed/verified.
   Mocking methods declared on non-public parent classes is not supported.
2. inside when() you don't call method on mock but on some other object.

```

Actually, this is because Mockito cannot mock a static method, in this case, getInstance(). 

### Step 02 : Use PowerMock

But I know PowerMock can mock static method, so I want switch to PowerMock instead.

```java
@RunWith(PowerMockRunner.class)
@PrepareForTest(FooManager.class)
public class FooManagerTest {

    @Test
    public void testSingleton(){
        FooManager mgr = Mockito.mock(FooManager.class);
        PowerMockito.mockStatic(FooManager.class);
        Mockito.when(FooManager.getInstance()).thenReturn(mgr);

        FooManager actual = FooManager.getInstance();
        assertEquals(mgr, actual);
    }

}
```

Yes, I succeed. But I have to mention that all the above code only succeed when your project  is not an Android project, just a pure java project. If we want to test Android code, we may have some another problems.

## Test Android Code
### Step 03 : test Andoid code with unit tests
You many think that Android is also written by Java. So you may write unit test in the `$module$/src/test` directory. 

But are you sure? Let's see an example of testing the code in the Android library  with JUnit Test.

```java
    @Test
    public void testAndroidCode(){
        instance.setArgu(argu);

        instance.doSomething();
        verify(argu).isCalled();
    }
```
However, you may get a failure:
`java.lang.NoClassDefFoundError: org/apache/http/cookie/Cookie`

Of course you may get a NoClassDefFoundError. But you may not find another class , for example, android/util/Log, android/content/Context ...

The reason why you get the NoClassDefFoundError is because JUnit is running on JVM, which means JUnit do not have the Android environment. 

Actually Android has a official solution to test with Android environment: [Instrumentation Test](https://developer.android.com/training/testing/unit-testing/instrumented-unit-tests.html). 

However, this is not what I really want. To run every instrumentation test, you have to build the whole project and push the apk to your phone or emulator. Yes, it's slow. It's not like JUnit, which can run on your computer(PC/Mac/Linux) and has no need of Android Environment.  As a result, JUnit test in your computer is much faster than instrumentation test.

Is there a solution that can involve Android environment and also run on the computer, which result in fast test ? Yes, there is. The answer is **Robolectric**!


### Step 04 : Robolectric



### Conclusion 01
I introduce use Robolectric to test Android code in a fast speed, and how to mock a singleton in java environment.

But I have to tell you, if you want to mock a Singleton in Android environment, you will fail.  And that part is what I will talk about in the later post.


