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

## 2. Why is it so hard to inject a Presenter?
The reason why this injection is hard is we already init a presenter inside Activity.onCreate(). That said, it hard to inject the presenter that we mocked to the Activity. 

Then what if we have a setter injection?
```java
public class DemoActivity extends Activity {
    private DemoPresenter presenter;
    @Override
    public void onCreate(Bundle b){
        super.onCreate(b);
        setContentView(R.layout.activity_demo);
        
        setPresenter(new DemoPresenter(); // that does not help.
        presenter.init();                
    }
    
    public void setPresenter(DemoPresenter presenter){
        this.presenter = presenter;
    }
}
```

You see the ` setPresenter(new DemoPresenter();` actually does not differ from the `presenter = new DemoPresenter()`. We still cannot make sure our solution could work on both scenarios :
* Production code use the `new DemoPresenter()`
* Test code use the mocked DemoPresenter

## 3. Approach 01: Dagger2
The first approach is the Dagger2. Dagger2 is a tool for injection, so you can separate the initilization and the use of one class. Of course, this is the aim we try to get in this post.

Here is what we should do.

* create a Component
```java
@Component
public interface SamplePresenterComponent {
    void inject(SampleActivity activity);
}
```
This Component actually builds a bridge that connect the user of one class and the generator of that class.

* create a Presenter (object generator)
```java
public class SamplePresenter {
    @Inject
    public SamplePresenter() {..}
}
```

* create the Activity (user of object)
```java
public class SampleActivity extends Activity implements ISampleView {

    @Inject SamplePresenter presenter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        DaggerSamplePresenterComponent.builder()
            .build()
            .inject(this);
    }
```














## 4. Approach 02: Setter Injection



## 5. Conclusion











