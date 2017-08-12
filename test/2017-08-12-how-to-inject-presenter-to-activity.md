## 1. Introduction

Here is a code snippet that you may face every day. 
```java
public class DemoActivity extends Activity {
    private DemoPresenter presenter;
    @Override
    public void onCreate(Bundle b){
        super.onCreate(b);
        setContentView(R.layout.activity_demo);
        
        presenter = new DemoPresenter();
        presenter.init();                
    }
}
```

This snippet should be good enough, if you don't write unit test. Now what if you want to unit test Activity, you may want to mock a presenter to isolate the dependency.

P.S. The isolation of dependency in unit test is super importatnt. 
* If a class contains logic about http, database, you will obviously not want your test failed just because the http timeout or database connection failure. If the unit test failed, the only reason should be code is broken, not the connection timeout or other external reasons. Then you should write a mock class and inject it to the target. 
* If a class is very complicated, which contains a bunch of different error and different logic (default, login, illegal user, eligible user, user that forget his/her password, ...),  it would be very difficult to reproduce all these scenario. But a mocked class would be a big help for you. You can use `when(mockObject.getUserType()).thenReturn(Login)` to mimic the logic scenario and all the other scenarios you want.

## 2. Approach 01: Dagger2




## 3. Approach 02: Setter Injection














