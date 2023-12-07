# A. What is RxJava



## 1. what is RxJava, in essence? 

We all know the observer design pattern, which is you register a listener/callback, and when something happens, the listener/callback will get called.  RxJava is actually an implementation of observer design pattern.

```kotlin
api.getUsers()
  .subscribe {users -> adapter.submitList(users) }
```


The side which send out data or event, we call it observable, or upstream

The side which receive data or event, we call it observer, subscriber, or downstream .



## 2. what’s more about RxJava

RxJava use a stream code pattern, which every operator is added between the upstream and downstream, such as : 

```kotlin
api.getUsers()
  .filter{ users.size > 0 }
  .map {users -> UserListModel(users)}
  .doOnNext {users -> analytics.logEvent("getUsers")}
  .subscribe {users -> adapter.submitList(users) }
```


The filter, map, doOnNext, …., we call them as operator.

Every operator can manipulate the data on the stream and will make our stream more flexible. Lucky for us, RxJava provides a loooot of built-in operator to help us to build powerful functionality.  Thread switching is one of them. 



## 3. Thread switching

Since Android 3.0, Android  no longer allow the IO operation on the main thread. So what we can do was : 

```kotlin
mainHandler = new Handler(Looper.getMainLooper())

new Thread(new Runnable() {
  @Override
  public void run() { 
    list = api.getUsers() // worker thread
    mainHandler.post (new Runnable { 
      @Override
      public void run() {
        adapter.submitList(list);  // main thread
      }
    });
  }
})
```


This is ugly to read, and boring to code for our dev as well. So RxJava uses an powerful way to easily switch thread

```kotlin
api.getUsers()
    .subscribeOn(Schedulers.io())
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe {users -> adapter.submitList(users) }
```

In this way, api.getUsers will get called in the IO thread, and the adapter.submitList will happen in the main thread. Easy to code, and also easy to read!



p.s. our c51 has done an extension (schedules() ) for this thread switching, so we don't have to write the same schedulers code again and again : 

```kotlin
api.getUsers()
    .schedules()
    .subscribe {users -> adapter.submitList(users) }
```

# B. Powerful  operators

map , filter , … is quite common in every modern language, so we will not explain them. 



## 1. get several endpoint before rendering

In the OfferList page, we would ask three endpoints to get the full data for this page, which are static, dynamic and getListCategory endpoints.



The key point here is to wait until three endpoints all send back their response. RxJava can handle it very easily. 


```kotlin
Flowable.zip(api.static(), api.dynamic(), api.categories())
    .schedules()
    .subscribe {users -> adapter.submitList(users) }
```

or

```kotlin
Flowable.combineLatest(api.static(), api.dynamic(), api.categories())
    .schedules()
    .subscribe {users -> adapter.submitList(users) }
```


zip and combineLatest could easily achieve this wait until have several results work. 


### difference between zip and combineLatest?

Let’s say I have a upstream which will send out ‘A' at time 10ms, ‘B’ at 500ms, and 'C’ at 2000ms.

And also I have another upstream which will sent out ‘1' , ‘2', ‘3’  at time 200ms, 400ms, 2500ms



If I am using Flowable.zip(upstream1, upstream2) then the output would be : 
```kotlin
time 200ms: A, 1
time 500ms: B, 2
time 2500ms: C, 3
```

→ that’s because zip will wait we have a full pair of data from both upstream, then it will send out the data to the downstream of zip.



If I am using Flowable.combineLatest(upstream1, upstream2) then the output would be : 
```kotlin
time 200ms: A, 1
time 400ms: A, 2
time 500ms: B, 2
time 2000ms: C, 2
time 2500ms: C, 3
```
→ that’s because combineLatest will send out a data once any upstream has new data. 





## 3. listen to network change and pull-down-refresh

We have a requirement that ask us to refresh the page when the network connectivity is changed, or when the user pull down to refresh the page manually. 

In this case, merge is the quite operator you are looking for.


```kotlin
// rxNetwork.listen() returns a Flowable<Boolean>
// pullToRefresh.listen() returns a Flowable<Boolean>
Flowable.merge(rxNetwork.listen(),  pullToRefresh.listen())
  .subscribe { isTriggered: Bool -> adapter.submitList(data)}
```


 merge  will trigger the downstream when any upstream changes. 



question for you:
* Is combineLatest okay to use in this scenario? 



## 3. “multiple action in a short time” bug

We have a “upload receipt” button in the app. When user tap this button, we will upload the receipt image, which is a huge-size picture to the backend.

```kotlin
btnUploadReceipt.clicks()
  .flatMap { uploadImage()}
  .subscribe {result -> ... }
```  

If a user taps a same button twice in a short time, our app will sent out two same pictures to the backend. How could we avoid this kind of scenario? 

: debounce could take care of this issue very quickly.   

```kotlin
btnUploadReceipt.clicks()
  .debounce(500, TimeUnit.MILLISECONDS)
  .flatMap { uploadImage()}
  .subscribe {result -> ... }
```

The code above will make sure no event is emitted before the 500ms elapses.  

During the 500ms, if a new event is emitted, the old event will be dropped and the 500ms timeout will re-start. 



Let’s say, 

* a user taps the button button four times,  at time 10ms, 110ms, 400ms,  then the uploadImage() will get called once, and actually happens at time 900ms (400 + 500 delay = 900)

* a user taps the button button four times,  at time 10ms, 110ms, 400ms, 600ms, then the uploadImage() will get called once, and actualy happens at time 1100ms (600 + 500 delay = 1100)



## 4. action after another action



map is change the data in the stream to a different data:

```kotlin
Observable.just(1, 2, 3)
  .map { it + 10}
  .subscribe { print("got $it")} //=> got 11; got 12; got 13
```


flatMap is change the data in the stream to a different stream: 
```kotlin
Observable.just(1, 2, 3)
  .flatMap { Observable.just(it + 10)}
  .subscribe { print("got $it")} 
  //=> got Observable; got Observable; got Observable
```
This way, flatMap is perfect to use some following-up actions. 



Here is an use case that we will need to tell the backend the user’s country information has changed, and once it’s done we need to refresh the cached user’s info locally as well. In this case, they two actions (talking to BE, and refresh data locally) is happening in sequence, we could use flatMap





p.s. If you are a former-JavaScript, or former-Angular user, then RxJS has no flatMap operator, but the functionality is still there, just the name has changed to mergeMap . 

# C. Flowable, Observable

If the upstream is sending out more data than what downstream can handle, a backpressure happens. 

As a receiver, it can drop the latest data, or it can buffer the coming data, or it can drop the old data and just use the latest data, ….  → these are the different strategies to handle backpressure. 



In the RxJava 1.0, an upstream must be a Observable type. 

But since RxJava 2.0+, to handle the backpressure more easier, Observable type has split type. oldObservable = Observable + Flowable

Observable is the upstreams that ignore the backpressure.

Flowable is the upstreams that will assign some backpressure strategy.



Here is an example to create a flowable, and you MUST assign a backpressure strategy to it: 

```kotlin
val upstream : Flowable<String> = 
  Flowable.create({ FlowableEmitter<String> emitter -> 
      emitter.onNext("abc")
      emitter.onNext("123")
  }, BackpressureStrategy.ERROR);
```


Back to our c51 project, which one to use?

: no strict rule. Because unlike the high throughput in the BE, our mobile has no so much data to handle for a stream in general. 

If you really want to pick one, then you could use Flowable , since it handles backpressures for us. 
But if you want to use Observable in most of scenarios,  no worries, no finger points here. 

# D. Error handling

## 1. catch error

If the connectivity is off, or the backend service is down, then the call below will fail and crash: 

```kotlin
api.getUsers()
  .subscribe { users -> renderPage(users) }
```



To handle more errors, the common approach is to catch the error in the subscribe block, and show some error views to notify the user that something is wrong.

```kotlin
api.getUsers()
  .subscribe (
    { users -> renderPage(users) },
    { error -> renderError(error)})
```


## 2. return default value

What if you just want to show an empty view once some error happens? 

: In this case, onErrorReturnItem is your friend. Once there is error, onErrorReturnItem(defaultValue) will return a default value to the subscriber. 


```kotlin
api.getUsers()
  .onErrorReturnItem(listOf<User>()) //return empty list if there is error
  .subscribe { users -> renderPage(users) } 
  
fun renderPage(users: List<User>) {
  if(users.isEmpty) { renderEmptyView() }
  else {adapter.submitList(users)}
}  
```


## 3. complete the stream

You can also choose another strategy: once the error happens, just complete the stream


```kotlin
api.getUsers()
  .onErrorComplete()
  .subscribe { users -> renderPage(users) }
```





## 4. listen to the error

you can add doOnError to listen to the error.

Note that, you still need to handle error using the previous approach, doOnError just listen to the error, it does not catch the errors. 


```kotlin
api.getUsers()
  .doOnError{ err -> analytics.logError(err) }
  .onErrorReturnItem( listOf<User>()) //return empty list if there is error
  .subscribe { users -> renderPage(users) }
```

#E. Subject and “hot observable”

This is quite a lot to cover in this topic if we need to explain these two topic in detail . Here we just introduce what we used the most. 



## 1. Subject

`Subject = Observable + Observer``, yes, it is a Observer, and also it is an observable.  So the next two lines of code are both legal

```kotlin
// send out data, it is a observable
subject.onNext(11) 
subject.subscribe {....}

// receiver data from another stream, it is a observer
anotherStream.subscribe(subject) 
```


The most common used subject is PublishSubject, you can create an PublishSubject object by  : 

```kotlin
val subject = PublishSubject.create<Int>()

subject.onNext(12)
subject.onError(RuntimeException())
subject.onComplete()

subject.subscribe {...}
```


The usage of BehaviorSubject, ReplaySubject is too complex to be covered in an introduction topic of RxJava. Hopefully we will cover these two and also the code observable; hot observable in the Android team’s internal Tech Share events. 



# F. RxBinding

RxJava, RxSwift is more like add extra power to Java and Swift

In the other hand, for the two most popular mobile platform, we have RxBinding and RxCocoa to handle the platform view event for Android and iOS, such as clicking a view,  pulling to refresh a page, …



## a world without RxBinding

Now let’s assume we have a requirement that we should persist the user’s input when the user is typing or the EditText has lost the focus. 

If we haven’t the RxBinding, this would how we do: 

```kotlin
val subject = PublishSubject.create<String>()

editText.addTextChangedListener(object : TextWatcher {
            override fun afterTextChanged(s: Editable?) {}

            override fun beforeTextChanged(s: CharSequence?, start: Int, count: Int, after: Int) {}

            override fun onTextChanged(s: CharSequence?, start: Int, before: Int, count: Int) {
                subject.onNext(s)
            }
        })
        
editText.setOnFocusChangteListener { view, isFocused ->
  subject.onNext(editText.getText().toString())
}        

subject
  .subscribe { inputText -> persist(inputText)}
```


## Now we had RxBinding 

if we have RxBinding, for the same scenario, the code would be much concise and readable: 

```kotlin
Flowable.merge(
      editText.textChangeEvents(), 
      editText.focusChanges().map { editText.text.toString()})
  .subscribe{ inputText -> persist(inputText)}
```


RxBinding added multiple extension methods to the views (more detail please see the table below), and these extension methods will return a Observable<T> out and let us to observe. 

In this example, the et.textChangeEvents() is a RxBinding extension method, to listen to the text change.

And the et.focusChanges() is used to listen to the focus change. 

## Typical RxBinding extension method
```kotlin
// listen to the attachToWindow, detachFromWindow event
fun View.attaches()
fun View.detaches()

fun View.clicks()
fun View.longClicks()

fun View.focusChanges()

// wait until all the layout is done (you can get the view's width and height then)
fun View.globalLayouts()
```


```kotlin
fun TextView.textChangeEvent() // return Observable<TextChangeEvent>
fun TextView.textChanges() // return Observable<String>

fun TextView.editorActionEvents() // action in the IME, such as: done, search, next, send, ...
```

```kotlin
fun CompoundButton.checkedChanged()

fun SeekBar.userChanges()

fun DrawerLayout.drawerOpen()

fun SwipeRefreshLayout.refreshes()
```



For the RecyclerView:
```kotlin
fun Adapter.dataChanges()

fun RecyclerView.childAttachStateChangeEvents()

fun RecyclerView.scrollStateChanges()
fun RecyclerView.scrollEvents()  
```




# G. C51’s extension

## thread switching

usage: 
```kotlin
userApiV3.updateCountry(country)
            .schedules()
```


source code: 
```kotlin
fun <T> Observable<T>.schedules(): Observable<T> =
        this.subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())

fun <T> Flowable<T>.schedules(): Flowable<T> =
        this.subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
```kotlin


## stateful data

LiveData will have the “re-commit” issue, and also has no state, so we suggest to use RxJava’s stream, instead of LiveData in the code.



by state , we mean the onLoading, onComplete, onError  . Let’s say

* we have to show a shimmer animation during the loading

* we have to refresh the list after complete

* we have to show an error view once there is error.

* no matter this work is successful or failed, we have to stop the shimmer animation

We can do so in the RxJava


```kotlin
        vm.batchRequest
            .aligns(this::onStartLoading, this::onLoaded)
            .subscribe { batchValue ->
                adapter.submitList(batchValue)
            }.clearedBy(disposables)
            
fun onStartLoading() {
  shimmerAnimation.start()
}            

fun onLoaded() {
  shimmerAnimation.stop()
}
```


the source code of align is just listen to the data in the stream: 

```kotlin
fun <T> Flowable<T>.aligns(onLoading: ()->Unit, onLoaded: ()->Unit) : Flowable<T> =
    this.doOnSubscribe { _ -> onLoading() }
        .doOnComplete(onLoaded)
        .doOnError { err -> onLoaded() }
```





## clear subscription when time is right

We have CompositeDisposable disposables in the BaseActivity, BaseFragment and BaseViewModel classes. So if you want to clear the subscription by the end of the page, you could just use clearBy()


```kotlin
  vm.fetchData(batchId,categoryId)
      .subscribe { data -> this.render(data) }
      .clearedBy(disposables)
```kotlin




source code of the clearBy: 
```kotlin
fun Disposable.clearBy(disposableQueue: CompositeDisposable) {
    disposableQueue.add(this)
}
```