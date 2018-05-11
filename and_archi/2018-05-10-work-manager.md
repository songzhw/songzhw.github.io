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

#### i. Worker
We extend `Worker` class and override the `doWork()` method. In this class we did what we need to do.

```kotlin
class PullWorker : Worker() {
    override fun doWork(): WorkerResult {
        // if the user accept "allowing pushing" in the preference page
        val isOkay = this.inputData.getBoolean("key_accept_bg_work", false)
        if(isOkay) {
            Thread.sleep(5000) // Mock the time consuming task

            val pulledResult = startPull()
            val output = Data.Builder().putString("key_pulled_result", pulledResult).build()
            outputData = output
            return WorkerResult.SUCCESS
        } else {
            return WorkerResult.FAILURE
        }
    }

    fun startPull() : String{
        return "szw [worker] pull messages from backend"
    }
}
```

#### ii. Wrap the Worker, and put it in a queue
We use `WorkRequest` to wrap the real worker, and then put it in the queue.

* `WorkRequest`: this is the real task in the queue. It has some extra attributes. 
    * ID (This ID is an UUID, to make sure the ID is unique)
    * When to execute this task
    * Constraints (like execute this task only when device is charing and online)
    * Execution chain. (WorkB can be executed only after the finish of WorkA)

* `WorkManager`: put the task(aka. `WorkReqeust`) into the queue.

```kotlin
class PullEngine {
    fun schedulePull(){
        //You could use "PeriodicWorkRequest.Builder" in Java
        val pullRequest = PeriodicWorkRequestBuilder<PullWorker>(24, TimeUnit.HOURS)
                .setInputData(
                    Data.Builder()
                        .putBoolean("key_accept_bg_work", true)
                        .build()
                )
                .build()
        
        WorkManager.getInstance().enqueue(pullRequest)
    }
}

```
### 3. explanation

1. `Worker` is the class that does the real job. But we noticed that doWork() does not parameters to get the input value, and it return `void`, which means it does not return a value. What if the task need some parameter to run, and also need to pass out some result?

Then it's time for our `Worker.getInputData()`, and `Woker.setOutputData()` methods.

2. The data we are talkinb about is the `Data` object, which could be created by `Data.Builder` class. Just like this:

```kotlin
val output = Data.Builder().putInt(key, 23).build()
```

3. We don't use `WorkRequest` class directly. Normally we would use its subclass: `OneTimeWorkRequest` or `PeriodicWorkRequest`.
Since the pulling job is a repeating job, so we used the `PeriodicWorkRequest` class to do that in our last code snippet.


## More features

### Deal with the result of your task
`WorkManager` provides a LiveData<WorkStatus> to tell you what happend. The `WorkStatus` object would tell you if the task if finished or not, and also you can get the output data from it, just using `WorkStatus.getOutputData()`.

By the way, since this is a `LiveData`, so you need a `LifecycleOwner` to observe this `LiveData`. Typically it would be the `AppcompatActivity`.

Now, still the pull example, we could print a log after we get the result from backend.

```kotlin
[PullEngine.kt]
class PullEngine {
    fun schedulePull(){
        val pullRequest = PeriodicWorkRequestBuilder<PullWorker>(24, TimeUnit.HOURS).build()
        WorkManager.getInstance().enqueue(pullRequest)

        // Add these two lines
        val pullRequestID = pullRequest.id
        MockedSp.pullId = pullRequestID.toString() // save the UUID
    }
}
```

```kotlin
class PullActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        // UUID is an implemented class of Serializable interface. And also could be converted to String, back and forth.
        val uuid = UUID.fromString(MockedSp.pullId)
        WorkManager.getInstance().getStatusById(uuid)
                .observe(this, Observer<WorkStatus> { status ->
                    if (status != null){
                        val pulledResult = status.outputData.getString("key_pulled_result", "")
                        println("szw Activity getResultFromBackend : $pulledResult")
                    }
                })
    }
}

```

One more thing to say, the observe() method is actullay `observe(LifecycleOwner, Observer<WorkStatus>)`


### 2. Input, Output

WorRequest: setInputData(Data)

Work: getInputData(), setOutputData(Data)

WorkStatus: getOutputData()


### 3. If the task finished, and your app is not started. What would happen?
This is a good question. The quick question is "nothing happens", especially your app will not be started by force.

```kotlin

```

