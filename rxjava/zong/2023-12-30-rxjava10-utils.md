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

# 五. debug时打日志

在debug时, 可能我们想知道何时rxjava流走到哪了, 这时就可以用以下几种方式来帮助我们: 

```kotlin
api.getUser()
  .doOnSubscribe {/* 下游注册了就会调用这. 相当于是冷流的开始了 */}
  .doOnCancel { /* 在disposable.dispose()时, 这个doOnCancel就会被调用到 */ }
  .doOnNext { item -> /* 每个数据都会走一次这里 */ }
  .doOnComplete { /* 在整个流完工了, 所有数据都发送完了都没错误的话, 这时就会调用这个方法*/ }
  .doOnError {err -> /* 出错了走这里. 注意, 并不会catch住error, 只是一个监听而已 */ }
  .doOnTerminate { /* 无论出错还是完结, 都调用这个方法*/ }
  .doFinally { /* 无论是complete, error, 还是cancel都会走一次这里. */ }
  .doAfterTerminate {/*无论出错还是完结, 都调用这个方法*/}
  . ... ....
```

注意: 当所有数据发送完了会走onComplete; 而当有错误时会走onError. <br/>
但有时有些操作, 要求无论是complte还是error, 都要执行. 如让"正在加载中..."的view给消失掉, 这时怎么弄呢?  <br/>
: 这时就可以利用`doFinally`. 这个监听方法无论是ocmplete还是error都会走一次的



备注: 对于Single这样类, 自然没有doOnNext, 但可以用`doOnSuccess`来代替的

# 六. 生命周期
简单的doOnNext (Single则是doOnSuccess), doOnError, doOnComplete就不讲了. 讲一点特别的: 

* **doOnSubscribe**: 在下游注册时被调用. 可以想像成"rxjava流的工作的开始"
* **doOnRequest**: 几乎等同于doOnSusbscribe. 二者也几乎是同时被调用的. 主要用于backpressure的debug, 其它情况很少用. => 这个只有Flowable有. 像Observable, Single等都没这个方法

* **doOnCancel**(Flowable中), ​**doOnDispose**(Single中叫这名): 这是在disposable.dipose()时被调用的. 一般是长时间执行工作的流才有. 普通的流发完数据就完了, 根本没留时间给我们dispose, 也就不会调用这两个方法

* **doOnTerminate**, ​**doAfterTerminate**: 这两个方法是保证无论error还是complte, 都会走到这两个方法. 区别在于一个是在terminate之前, 另一个是在terminate之后.
  * 以single成功的场景为例, 日志就是: doOnSubscribe -> doOnSuccess -> doOnTerminate -> 下游subscribe中收到数据 -> doAfterTerminated
  * 注意, 若下游没有catch error, 那出错时就只有doOnTerminate被调用. 因为app已经crash了, doAfterTerminated就没机会被调用app就已经结束了
  * 注意, disposable被cancel时, 不会走到这两个方法来!

* **doFinally** : 它的时机是类似于doAfterTerminate, 即在下游收到最后一个数据, 或下游收到error后被调用.
  * 注意名字不是叫doOnFially.
  * 同样在下游没有catch住error时, 也只有doOnTerminate被调用. 因为app已经crash了, 所以doAfterTerminate与doFinally全不会被调用到.
  * 另外一个显著特点就是, 当disposable.dispose时, 会调用doOnCancel, 也会调用doFinally. 但不会调用doOnTerminate与doAfterTerminate.



# 七. 如何把带callback的旧代码转成RxJava流? 
比如说我们有一个两个方法, 它们是依次调用的, 而且都是老式的callback方式: 

```kotlin
api.getUser(object: Callback {
    override fun onSucc(user: User) { 
        api.postSetting(user.id, settings, object: Callback {
            override fun onSucc(result: String) { ... }
            override fun onFaile(error: Exception) { ... }
        })
    }
    override fun onFaile(error: Exception) { ... }
}
```

现在我们有一个新需求, 在拿到user信息并post此用户的设置到后台后, 若postSetting成功, 那我们就要通知一个第三方此事件, 你可以理解为埋点. 老式的做法肯定又是嵌套了, 也就是人们常说的callback地狱了. 

```kotlin
api.getUser(object: Callback {
    override fun onSucc(user: User) { 
        api.postSetting(user.id, settings, object: Callback {
            override fun onSucc(result: String) { 
                api.notifyXX(settings, object: Callback {
                    override fun onSucc(result: String) { ... }
                    override fun onFaile(error: Exception) { ... }
                })
            }
            override fun onFaile(error: Exception) { ... }
        })
    }
    override fun onFaile(error: Exception) { ... }
}
```

我们的代码仓库要是很老, 这样的callback回调式异步代码肯定不少见. 这时我们可以在需求时把它们转为RxJava流. 转化的方法如下, 我仅以api.getUser为例哦

```kotlin
// 老方法
fun getUser(callback) {.....}

// 新加一个getUser$方法, `$`后缀代表一个Stream. (当然Kotlin中,`$`不能做为名字. 这里仅为了更易理解)
fun getUser$() : Single<User>{
    return Single.create<User> { emitter -> 
        this.getUser(object: Callback{
            override fun onSucc(user: User) { 
                emitter.onSuccess(result)
             }
            override fun onFaile(error: Exception) {
                emitter.onError(error)
             }
        })
    }
}
```


这样一来, 当我们想要依次执行三个api时, 就再也不用callback hell了, 可以直接用: 

```kotlin
// 新式做法
api.getUser$()
    .flatMap {user -> api.postSetting$(user.id)}
    .flatMap {setting -> api.notifyXX$(settings)}
```

这个代码是不是要比callback hell要好看得多, 也不容易看错了. 


## 小技巧
若是第三个技巧, 又要用到user数据, 又要用到setting数据, 那要怎么办?

: 好办, 在第二个流里加一个map, 把user数据也带上就行了, 即: 

```kotlin
// 新式做法
api.getUser$()
    .flatMap {user -> api.postSetting$(user.id)
                         .map {setting -> Pair(user, setting)}}
    .flatMap {pair -> api.notifyXX$(pair.first.userId, pair.second)}
```

