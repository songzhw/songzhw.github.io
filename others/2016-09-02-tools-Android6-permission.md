Since the Android 6.0 is published, the dynamic permission request system is a good news for the user. However, it means we developer will code more, even more biolerplate. 

Is there a better way to code more easier? Yes, `Permission6` is here to rescue us.

Before I introduce the `Permission6` to you, I have to say there are many libraries to do such a job. Most of them requires you to subclass your Activity to their Activity. This way, their Activity can do the `Activity.onRequestPermissionsResult()` and `Activity.requestPermissions()` job for you. 

However we ususaly have our own base Activity as a parent for all our own Activity, and Java only allows sinlge inheritance, so these libraries I mentioned before can't meet our requirement. 





