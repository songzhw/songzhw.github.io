Recently I had a big problem : How to test singleton in a Service? I finally managed to fix this problem. But I think the process of how to solve this is a great opportunity to explain unit test clearly. So I write this post. 

### Our Service

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
