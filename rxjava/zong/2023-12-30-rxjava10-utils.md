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
fun Disposable.clearedBy(disposableQueue: CompositeDisposable) {
    disposableQueue.add(this)
}

class BaseActivity: AppCompatActivity() {
    protected val disposables = new CompositeDisposable();

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
