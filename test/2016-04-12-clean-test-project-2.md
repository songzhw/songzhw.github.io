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




```java

```

