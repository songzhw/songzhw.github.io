# How to build a clean, test project for your app (2)?


## 1. Shortcoming of the last way
I introduced a ```Config.isDebugFroTest``` way to differ the test codes and develop codes. Now we can easily switch the ```isDebugForTest``` to on or off. 

However, this way has a obvious shortcoming. It combines the develop codes and test codes in one same files, which lays in the development directory. This defective codes is this :


```java
[Activity]
    private void startRequest() {
    	if(isDebugForTest){
    		onResponsed(MockApiRepo.API_USER);
    	} else{
	    	BaseRequest req = new BaseRequest();
	        String url = "https://api.github.com/users/songzhw";
	        req.startRequest(url, this);
        }
    }

    @Override
    public void onResponsed(String resp) {
        final User user = new Gson().fromJson(resp, User.class);
        tv.setText(user.name);
    }
```
What I need is the separation of the develop code and the test code. I want that Activity only has the 

```java
[Activity]
    private void startRequest() {
//    	if(isDebugForTest){
//    		onResponsed(MockApiRepo.API_USER);
//    	} else{
	    	BaseRequest req = new BaseRequest();
	        String url = "https://api.github.com/users/songzhw";
	        req.startRequest(url, this);
//        }
    }
```
part. And the test codes can also easily turn on the test switch.

## 2. The process of my continuous trying

To manage the previous demand, I thought about Dagger2, AndroidJUnitRunner, and other ways. These ways actually can work well. But they are a little complex than I expected. I want a clean test framework, with a easier expandation for the programmer. 

## 3. My final solution

Since I have to include the develop code in the Activity, I have to use some little tricks. 

```java
// app/src/main/.../MainActivity.java

public class MainActivity extends Activity{
	public BaseRequest req; 

	public void onCreate(Bundle b){
		// ... 
		req = new BaseRequest();
    	// btn.setOnClickListner{ v -> startRequest() }
    }

    private void startRequest() {
        String url = "https://api.github.com/users/songzhw";
        req.startRequest(url, this);        
    }

    @Override
    public void onResponsed(String resp) {
        final User user = new Gson().fromJson(resp, User.class);
        tv.setText(user.name);
    }
}
```

And how do we turn on the debug switch on the test codes？

(1). Firstly, I have to mock a Request. So I write a new class which is the child of ```BaseRequest```

```java
public class MockRequest extends BaseRequest {
    @Override
    public void startRequest(String url, IRespListener alistener) {
        if(alistener != null){
            alistener.onResponsed(MockApiRepo.API_USER);
        }
    }
}

```
In this fake Request, I do not actually visit the API through http. I directly jump to the listener of this request. So we can say we manage to mock the http process.


(2). Then I only have to assign this mocked Request to our Activity. Now, it is perfect.

```java
//app/src/androidTest/.../UserTest.java

@Rule
public ActivityTestRule<MainActivity> activityRule = new ActivityTestRule<MainActivity>(MainActivity.class);


@Test
public void testFackData(){
   
    MainActivity actv = MainActivity.getActivity();
    actv.req = new MockRequest();

    onView(withId(R.id.fab))
                .perform(click());

    onView(withId(R.id.tv_main))
            .check(matches(withText("songzhw")));
        
}

```
## 3. Conclusion
Actullay, what I have done is not so big. All I did just obey the principle

```
[Design Principle]

Programm to an interface, not an implementation.
```

I use the BaseRequest as a member of our Activity. And I use a child class of BaseRequest to replace it in the test codes. In this way, I easily decoupled the release codes and the test codes.


p.s. : A question
How about an app which uses Retrofit + OkHttp? How can I mock this app in the test codes?

You can think about it and welcome to try. My tip is also :

```
Programe to an interface, not an implementation!
``
