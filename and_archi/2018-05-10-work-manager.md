Google just released two new Architecture Component libraries on Google I/O 2018: Naviagation and WorkManager. Now I will talk a little bit more about WorkManager.

## One word about WorkManager
WorkManager is an component to help you execute tasks on the background, even if your app is not started at that time.

### 1. Why don't we just use AlarmManger?
As we said, the WorkManager will execute tasks even your app is killed or the deviced rebooted before. It sounds like a job registed in the AlarmManager. 
Yes, it is. Schedulering tasks is a delicate thing. Above Android 5.0, you can use `JobSchedulre`. Below Android 5.0, you can use `AlarmManager`. You can also use Firbase's `JobDispatcher`.  That is, in fact, what WorkManager does in the background. It might use one of these three approaches to help you achieve your goal.

### 2. Should we stop using AsyncTask, RxJava?
This is an excellent question. In fact, WorkManager is not a replacement for AsyncTask/RxJava. 

I quote: 
`WorkManager is intended for tasks that require a guarantee that the system will run them even if the app exits, like uploading app data to a server. It is not intended for in-process background work that can safely be terminated if the app process goes away;`
For situations like that, we recommend using [ThreadPools](https://developer.android.com/training/multiple-threads/create-threadpool#ThreadPool).

## Sample

### 1. adding WorkManager library

```groovy
[kotlin]
implementation "android.arch.work:work-runtime-ktx:1.0.0-alpha01"

[java]
implementation "android.arch.work:work-runtime:1.0.0-alpha01"
```

### 2. An example of pull
Back to 2012, I was doing an e-commerce project in China. We don't have a push service library back then, since Google exited China. And we had an app to release really soon. So instead of integrating a push library, we tried to use "Pull" strategy. We will try to pull the lasted recommeded goods from backend every 24 hours. 

We need to do two things: One, do the pulling; Two, execute this task, or enqueue this task.

