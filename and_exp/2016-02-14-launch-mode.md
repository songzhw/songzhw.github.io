#Activity Launch Mode

# adb shell dumpsys activity
This command can give you a clear view of each task, like how many tasks do you have, which activities are in which task. 

Now I have one picture of this command's output.

![](/imgs/20160214_01.png)

From this output, we can see, now ,we have two Task (#103, #102).

Task #103 : affinity = "cn.six.task2", size = 3 (it has three activities in it)<br/>
	-- Activity One <br/>
	-- Activity Three<br/>
	-- ActivityTwo<br/>

Task #102 : affinity = "cn.six.adv", size = 1<br/>
	-- Activity One<br/>
	
With this "adb shell dumpsys activity" command, we can explor the LaunchMode easier.	s

# Default

The Default system always creates a new instance of the activity in the target task and routes the intent to it.

"Default" is also the default mode when you assign no launch mode to an Activity, which means if you do not assign any launch mode to a Activity, then that Activity will be the "default" launch mode.


# SingleTop

If an instance of the "SingleTop" activity already exists at the top of the target task, the system routes the intent to that instance through a call to itsonNewIntent() method, rather than creating a new instance of the activity.


p.s. It's **NOT** clear_top !!!

# SingleTask

This is the most tricky one, and I have to spend much more time to explain it. And examples are a good way to explain this complex launch mode.

## 1. A(Default) -> B(singleTask)
We have two Activity, A and B, and A is default mode, B is singleTask mode. Now A jump to B. 

The manifest is like this:

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="cn.six.adv" >
	<Activity android:name=".A"/>
	<Activity android:name=".B" android:launchMode="singleTask"/>
</manifest>
```


Android documents says, "The system(SingleTask) creates the activity at the root of a new task and routes the intent to it".  So it must be like this, right?

Task 1  | Task 2
 :-------------------------:|:-------------------------:
 A  | B


But actually, when we run the "adb shell dumpsys  activity", we found out that B is strangely in the same task with A.

Task 1  | Task 2
 :-------------------------:|:-------------------------:
 A <br/> B | (null)

This is a little complex to explain, because it involves the ```android:taskAffinity``` attribute. I will explain this attribute later.


## 2. A(Default) -> B(singleTask) : B has a taskAffinity attribute
The manifest is like this:

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="cn.six.adv" >
	<activity android:name=".A"/>
	<activity android:name=".B" android:launchMode="singleTask" android:taskAffinity="task2"/>
</manifest>
```
I have to tell you, the result of A starting B is different. The tasks are like this:

Task 1  | Task 2
 :-------------------------:|:-------------------------:
 A  | B

The only difference between this and the previous example is the "android:taskAffinity". When you announced no affinity, then activity has a default affinity value : their package name. In this case, the default affinity value is "cn.six.adv". 

When A starts B, and B's launch mode is singleTask. So B **intents to** to create a new task. But it will create a new task as long as B's taskAffinity is different from A's taskAffinity. 

So this explains the previous examle: why A and B are in the same task. Because their have a same taskAffinity.

The logic is like this:

```
A --> B

  if(taskAffinity is the same) { 
  	A and B are in the same Task
  }
  else { 
  	B is in other new task, whose affinity is B's affinity
  }

```
And Coming to this example, A starts B, and B's launchMode is "singleTask", and B's taskAffinity is not "cn.six.adv". So B will really create a new task, and B is in this new task. 

Task 1 (affinity="cn.six.adv") | Task 2 (affinity="task2")
 :-------------------------:|:-------------------------:
 A  | B

## 3. A(default) -> B(singleTask) -> C(singleTask)


# SingleInstance


# Conclusion

Default  |   SingleTop |  SingleTask                  |  SingleInstance 
:-------------------------:|:-------------------------:|:-------------------------:|:-------------------------:
__  |  __  |  __  |  __