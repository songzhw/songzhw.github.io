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

"Default" is also the default mode when you assign no launch mode to an Activity.


# SingleTop


# SingleTask


# SingleInstance


# Conclusion

Default  |   SingleTop |  SingleTask                  |  SingleInstance 
:-------------------------:|:-------------------------:|:-------------------------:|:-------------------------:
__  |  __  |  __  |  __