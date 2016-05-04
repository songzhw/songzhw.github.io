# Fix Memory Leak


## Problem
Memory leak is a great issue in Android development since memory in Android is precious and limited. So it is very important to review your project codes now and then. 

Last month, I review my project. First init GC, then dump java heap. And I did find a memory leak.  

My project starts with a ```ProgressActivity``` to show a ProgressBar.  Then when I get the data from server, this ```ProgressActivity``` will call ```java this.finish();```, and jump to the ```MainActivity```. 

However, when my phone shows the ```MainActivity```, but I still can see a object of ```ProgressActivity``` in my memory. Why? Shouldn't it be finished now?

Then I looked at the reference tree, and found out ```SecurityHelper``` holded the reference of ```ProgressActivity```.   This  ```SecurityHelper``` is from a jar package my company developed. Then it hits me that maybe it's because the Singleton. 

## Singleton causes memory leak?



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