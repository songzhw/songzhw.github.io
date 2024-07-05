RxJava是十分强大的存在. 它的一大特别就是流式结构, 或叫串式结构, 就是类似这样的: 
```kotlin
Observable.just(1)
  .map {...}
  .filter {...}
  .switchMap {...}
  .doOnNext { .... }
  .doOnError {.... }
  .subscribe {....}
```
所以我们下面的很多对RxJava的加强, 全都是维持了这种串式结构. 

# 一. 对"线程切换"的扩展
我们经常都是要去工作线程做工作 (如后台接口的请求, 数据库的访问, 本地文件的操作, ....).  而把结果刷新到UI上, 一般又是在主线程上.  所以我们经常写subscribeOn与observeOn, 有点累. 

所以我们有了下面的的扩展
```kotlin

fun <T> Observable<T>.schedules(): Observable<T> =
        this.subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())

fun <T> Flowable<T>.schedules(): Flowable<T> =
        this.subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())

fun Completable.schedules(): Completable =
    this.subscribeOn(Schedulers.io())
        .observeOn(AndroidSchedulers.mainThread())

fun <T> Maybe<T>.schedules(): Maybe<T> =
    this.subscribeOn(Schedulers.io())
        .observeOn(AndroidSchedulers.mainThread())

fun <T> Single<T>.schedules(): Single<T> =
    this.subscribeOn(Schedulers.io())
        .observeOn(AndroidSchedulers.mainThread())

```

这样使用起来只要用: 
```kotlin
api.getUsers() 
  .schedulers()
  .subscribe{ ... }
```

# 二. 对"适时取消订阅"的扩展
`suscribe`方法返回的Disposable是要在页面退出时给注销掉的, 不然容易内存泄露.

不过给每个Activity, Fragment, ViewModel都加一个CompositeDisposable, 然后重写它们的onDestory, onDestoryView, onClear有点太累了, 重复代码太多<br/>
所以我们有了这样的代码:
```kotlin
fun Disposable.clearBy(disposableQueue: CompositeDisposable) {
    disposableQueue.add(this)
}

fun Disposable.clearBy(disposableQueue: SerialDisposable) {
    disposableQueue.set(this)
}




class BaseActivity: AppCompatActivity() {
    protected val disposables = new CompositeDisposable();

    @Override fun onStop() {
        disposables.clear();
        super.onStop();
    }

    @Override fun onDestroy() {
        disposables.dispose();
        super.onDestroy();
    }
}
// 对BaseFragment, BaseViewModel都可以这样用
```


这样我们的RxJava流就只样这样写, 就能保证在页面退出时会清除掉Disposables
```kotlin
api.getUsers() 
  .schedulers()
  .subscribe{ ... }
  .clearBy(disposables)
```

# 三. 对'状态'的扩展
即做一个工作, 如load data, 它有一个loading过程, 然后有loaded的回调(也可能是出错的回调). 所以这时我们做这样一个扩展: 
```kotlin

fun <T> Observable<T>.aligns(onLoading: ()->Unit, onLoaded: ()->Unit) : Observable<T> =
    this.doOnSubscribe { _ -> onLoading() }
        .doOnComplete(onLoaded)
        .doOnError { err -> onLoaded() }

fun <T> Flowable<T>.aligns(onLoading: ()->Unit, onLoaded: ()->Unit) : Flowable<T> =
    this.doOnSubscribe { _ -> onLoading() }
        .doOnComplete(onLoaded)
        .doOnError { err -> onLoaded() }

// 只有onError, onComplete.  (没有onNext)
fun <T> Completable.aligns(onLoading: ()->Unit, onLoaded: ()->Unit) : Completable =
    this.doOnSubscribe { _ -> onLoading() }
        .doOnComplete(onLoaded)
        .doOnError { err -> onLoaded() }

// Maybe是只有onSuccess, onError, onComplete  (没有onNext)
fun <T> Maybe<T>.aligns(onLoading: ()->Unit, onLoaded: ()->Unit) : Maybe<T> =
    this.doOnSubscribe { _ -> onLoading() }
        // .doOnComplete(onLoaded)  // Maybe虽然有doOnComplete, 但不管成功有数据还是失败有错误, 都不走doOnComplete
        .doOnSuccess { _ -> onLoaded() } // 二选一, 有数据了就走这
        .doOnError { err -> onLoaded() } // 二选一, 有错误了就走这

// single只有onSuccess, onError    (没有onNext)
// 注意, Single类没有doOnComplete方法
fun <T> Single<T>.aligns(onLoading: ()->Unit, onLoaded: ()->Unit) : Single<T> =
    this.doOnSubscribe { _ -> onLoading() }
        .doOnSuccess { _ -> onLoaded() } // 二选一, 有数据了就走这
        .doOnError { err -> onLoaded() } // 二选一, 有错误了就走这
```

所以我们可以这样做: 
```kotlin
  api.getUsers()
      .schedules()
      .aligns(this::showShimmer, this::hideShimmer)
      .subscribe{ jutes -> adapter.submitList(jutes) }
      .clearBy(disposables)

fun showShimmer() {
  shimmerLayout.isVisible = true
  shimmerLayout.start()
}      

fun hideShimmer() {
  shimmerLayout.stop()
  shimmerLayout.isVisible = false
}
```


# 四, 把冷流变为热流
在RxJS中这很容易, 因为它有`publishBehavior`, `publishReplay`, `publishLast`等操作符把一个cold observable变成BehaviorSubject, ReplaySubject与PublishSubject.

但不幸地的, RxJava中没有这几个操作符. 所以我现在加一个扩展, 就是达到`publishLast`的功能
```kotlin
fun <T> Observable<T>.makeHot(): Observable<T> {
    val bridge = PublishSubject.create<T>()
    this.subscribe(bridge)
    return bridge
}
```

这样就把一个冷流变成了PublishSubject这样一个热流. 


当然, 要是想达到RxJS中的效果, 那我们可以改成这样的扩展: 
```kotlin

fun <T> Observable<T>.publishLast(): Observable<T> {
    val bridge = PublishSubject.create<T>()
    this.subscribe(bridge)
    return bridge
}

fun <T> Observable<T>.publishBehavior(): Observable<T> {
    val bridge = BehaviorSubject.create<T>() // 没有默认值, 就会自动缓存一个
    this.subscribe(bridge)
    return bridge
}

fun <T> Observable<T>.publishReplay(): Observable<T> {
    val bridge = ReplaySubject.create<T>()
    this.subscribe(bridge)
    return bridge
}
```