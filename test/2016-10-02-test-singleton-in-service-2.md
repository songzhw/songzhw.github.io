## dilemma 
Last post introduce how to test a Singleton(**PowerMock**!), and how to unit test Android code (**Robolectric**!). Now we use a singleton in a Service class. And we would like to test it. So it should be easy, right?



### try 01: Combine PowerMock and Robolectric (1)
```java
// src/PushService
// [PushService.java]
public class PushService extends Service {
    public void onMessageReceived(String id, Bundle data){
        FooManager.getInstance().receivedMsg(data);
    }
}
```
So I combine PowerMock and Robolectric and write a test case: 

```java
// test/PushServiceTest
@RunWith(RobolectricTestRunner.class)
// @RunWith(PowerMockRunner.class)
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
Sonn, I find a dilemma, I can use `@RunWith(RobolectricTestRunner.class)`, or I can use `@RunWith(PowerMockRunner.class)`. But I cannot use them both! Once I can use them both, it means that I can choose to use Robolectric or to use  PowerMock, but I can not combine them.

### try 02: Combine PowerMock and Robolectric (2)
So I googled and wished to search a solution. Thank goodness, I did find a solution. This solution is posted by Robolectric, and the address is here: [https://github.com/robolectric/robolectric/wiki/Using-PowerMock](https://github.com/robolectric/robolectric/wiki/Using-PowerMock)

The post suggest me to add such head:

```java
@RunWith(RobolectricTestRunner.class)
@Config(constants = BuildConfig.class, sdk = 21)
@PowerMockIgnore({ "org.mockito.*", "org.robolectric.*", "android.*" })
@PrepareForTest(Static.class)
public class DeckardActivityTest {
    ...
}
```

I did what the post said, and run the test. But the test failed again. This time, the error message is :
```
com.thoughtworks.xstream.converters.ConversionException: Cannot convert type org.apache.tools.ant.Project to type org.apache.tools.ant.Project
---- Debugging information ----
```
<P><P>
So I contiued to search. This time I found an issue on [github.com/Robolectric](https://github.com/robolectric/robolectric/pull/2390) . In this issue, some people said:

```
Realistically we're not going to get to this till October, but we do
welcome contributions if you want to take a stab at fixing it yourself.

The best work around it to simply make your code testable then you don't
need to mock statics
```
<P><P>

Now I understand there is no such solution for using PowerMock and Robolectric at the same time. The solution may be out in 2016 October, but now (2016, September) I have to test the singleton in Service. How should I do?

### try 03: decouple Singleton



