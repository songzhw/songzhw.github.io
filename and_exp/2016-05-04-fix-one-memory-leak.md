# Fix Memory Leak


## Problem
Memory leak is a great issue in Android development since memory in Android is precious and limited. So it is very important to review your project codes now and then. 

Last month, I review my project. First init GC, then dump java heap. And I did find a memory leak.  

My project starts with a ```ProgressActivity``` to show a ProgressBar.  Then when I get the data from server, this ```ProgressActivity``` will call ```this.finish();```, and jump to the ```MainActivity```. 

However, when my phone shows the ```MainActivity```, but I still can see a object of ```ProgressActivity``` in my memory. Why? Shouldn't it be finished now?

Then I looked at the reference tree, and found out ```SecurityHelper``` holded the reference of ```ProgressActivity```.   This  ```SecurityHelper``` is from a jar package my company developed. Suddenly it hits me that maybe it's because the Singleton. 

## Singleton causes memory leak?

```java
private static SecurityHelper instance = null;

public static SecurityHelper getInstance(Context ctx){
	if(instance == null) {
		instance = new SecurityHelper(ctx);
	}
	return instance;
}
```

This is a typical Singleton.  However, if you do not pay extra attention, you may leak memory here. 

For example, if you use ``` SecurityHelper.getInstance(thisActivity); ``` , then your Activity reference will be holded by this static object ```instance```. 

Static objects have very long life cycle, which means any referenced holded by them will not get released. 

In this case, a better way should be ``` SecurityHelper.getInstance(thisActivity.getApplicationContext()); ```.  Since the Application reference is already existing all the time, there is no leak with it. In this way, the last Activity, and all the reference that that Activity holds, can be freed!


**Conclusion**: The Singleton itself will not cause memory leak. But it may with your careless use Context.  Remeber, to avoid memory leak, it should be careful when handling the static object. 


## Solution
The wrong code:

```java
securityHelper = new SecurityHelper(this);
```

The right code:

```java
securityHelper = new SecurityHelper(this.getApplicationContext());
```

After I modified the code, now the memory leak vanished!