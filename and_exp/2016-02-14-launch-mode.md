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
	
With this "adb shell dumpsys activity" command, we can explor the LaunchMode easier....

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
 B <br/> A | (null)

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

## 3. A(default) -> B(singleTask) -> C(singleTask) -> B(singleTask)

The manifest is like this:

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="cn.six.adv" >
	<activity android:name=".A"/>
	<activity android:name=".B" android:launchMode="singleTask" android:taskAffinity="task2"/>
	<activity android:name=".C" android:launchMode="singleTask" android:taskAffinity="task2"/>
</manifest>
```

(1). A -> B

Task 1 (affinity="cn.six.adv") | Task 2 (affinity="task2")
 :-------------------------:|:-------------------------:
 A  | B

(2) A -> B -> C

Since C's affinity is "task2", which is already the affinity of one task, so C will be put in the Task 2.

Task 1 (affinity="cn.six.adv") | Task 2 (affinity="task2")
 :-------------------------:|:-------------------------:
 A  | C <br/> B

(3) A -> B -> C -> B

Firstly, I want to show you the result

Task 1 (affinity="cn.six.adv") | Task 2 (affinity="task2")
 :-------------------------:|:-------------------------:
 A  | B

Weird, right? Where is C?

Let me explain it. C->B.  B is singleTask and its affinity is "task2", then the system find a task whose affinity is task2 and will put B in this Task 2. However, an instance of B is already existing in the Task 2. So the system will invoke the existing instance of B by give it a flag **CLEAR_TOP**. This is why C is not existing anymore. 

## 4. Conclusion of SingleTask

```java
	if( found a task whose affinity == Activity's affinity){
		if(an instance of this Activity already exists in this task){
			start this activity with a CLEAR_TOP flag
		} else {
			create a new instance of this activity in this task
		}
	} else { // do not find a proper task
		create a new task, whose affinity is this activity's affinity
		create a new instance of this activity in this new task
	}

```

# SingleInstance

SingleInstance is much easier than SingleTask.

Once one Activity is SingleInstance, this Activity will definitly in one new task, and this task can only hold this one Activity and no other Activities any more.

Let me show some examples.

### 1. A(default) --> B(singleInstance) --> C(default)

(1). A -> B

Task 1  | Task 2
 :-------------------------:|:-------------------------:
 A  | B

 (2). A -> B -> C

 A "singleInstance" activity permits no other activities to be part of its task. It's the only activity in the task. If it starts another activity, that activity is assigned to a different task — as if **FLAG_ACTIVITY_NEW_TASK** was in the intent.

Since B requires a quiet task which can only take him, so C will be added a FLAG_ACTIVITY_NEW_TASK flag. So C(default) becomes C(singleTask).

So the result is like this:

Task 1  | Task 2
 :-------------------------:|:-------------------------:
 C <br/> A  | B



p.s, if the process is "A(default) --> B(singleTask) --> C(default)", then the result is like this.

Task 1  | Task 2
 :-------------------------:|:-------------------------:
 A  | C <br/> B

### 2. A(default) --> B(singleInstance) --> B(singleInstance)

Task 1  | Task 2
 :-------------------------:|:-------------------------:
  A  | B

`B->B`, the result would be only one B in the task 2. And the B's `onNewIntent` will get called. 


### 3. A(default) --> B(singleInstance)
Note that even A and B have the same affinity, but now they will stay in two different tasks. 
These two tasks would have a different taskID, but their task affinity would be the same 


### 4. Note
1. if your `SingleTask` Activity has the default affinity, no matter you add the `NEW_TASK` flag or not, this activity would still lie in the main stack.

2. If you `SingleTask` Activity has its own affinity, then no matter you add the `NEW_TASK` flag or not, this activity would still lie in a different stack from the main stack.
  -> this applies to `SingleInstance` as well.

# SingleInstancePerTask
Since Api 31 (Android 12), a new launch mode `SingleInstancePerTask` was coined. Let me show you two examples, and you will get to know it.

## Background
* MainActivity (no affinity, no launchMode)
* A (affinity = ca.six.t2, no launchMode)
* B (affinity = ca.six.t2, launchMode = SingleInstancePerTask)
* C (affinity = ca.six.t3, launchMode = SingleInstancePerTask)

## Experiment 1
`Main -> A -> B`

If B's launchMode is `singleTask`, then B will find out there already is a task it needs (ca.six.t2), so the tasks would be:

Task 1  | Task 2
 :-------------------------:|:-------------------------:
Main |  B <br/> A 


But now B's luanch mode is `singleInstancePerTask`, which would **make your activity as a root of a task**, so B would never stay with the same task with A since A is already the root of task2.

Task 1  | Task 2 | Task3
 :----:|:----:|:----:
 Main | A  | B

## Experiment 2
`Main -> A -> B -> C -> B`

 The tasks would be:
 Task 1  | Task 2 | Task3 | Task4
 :----:|:----:|:----:|:----:
 Main | A  | B | C

 And the second B actually is not created, in fact. It just called B's `onNewIntent()` method as this new launchMode will keep a single instance per task

 You may ask, but two B does not result in two task, what happened?

## Experiment 3
`Main -> A -> B -> C --FLAG_MULTIPLE_TASK--> B`

We this time just add one more line of code:

```kotlin
            val intent = Intent(this, B::class.java)
            intent.addFlags(Intent.FLAG_ACTIVITY_MULTIPLE_TASK)
            startActivity(intent)
```

Now the tasks are:

 Task 1  | Task 2 | Task3 | Task4 | Task 5
 :----:|:----:|:----:|:----:|:----:
 Main | A  | B | C | B

 ## Conclusion

 * `SingleInstance`'s task can only have one Activity instance
   * but `SingleInstancePerTask` is different, the task can have many activities, just need to make sure the `SingleInstancePerTask` activity is the root activity of this task
 
 * `SingleTask` can be and not be a root of one task, but `SingleInstancePerTask` must be the root

 * `SingleTask` also can have one instance among multiple tasks, but in conjuction with the `FLAG_ACTIVITY_MULTIPLE_TASK`, `SingleInstancePerTask` would make your activity appear in multiple tasks

