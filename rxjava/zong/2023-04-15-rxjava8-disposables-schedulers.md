# "出门时关上门"
在以前学习Observer设计模式时, 我们就知道, 不能光是addListener. 在合适的时机, 我们也要removeListener. 不然就可能会因为对象还在而导致一些重要的类(如Activity, Fragment, ..)得不到GC释放, 从而造成内存泄露.  这就像是开了门出去, 那出去时要随手带上手是一样的. 

在RxJava中, 这个工作就是由`Disposable`来完成的. 
* 我们每次`subscribe`就会返回一个`Disposable`对象
* 在时机合适, 如页面被销毁时, 就调用`Disposable#dispose()`方法, 来释放掉这种订阅. 

# 有多个Dispose时
当然, 若我们一个Activity中有多个流以及它们的subscribe订阅, 那我们就要在onDestory中一一来dispose. 好累. 

所以更好的办法, 是使用 `CompositeDisposables`.  它内存有一个Set<Disposable>来存这些dispose. <br/>
到了要释放时, 就对这个set做for-each, 来一一释放这些dispose. 

举例来说, 一个Activity就是: 
```kotlin
class BaseActivity : Activity() {
  protected val disposables = CompositeDisposables()

  override fun onDestroy() {
    disposables.dispose()
    super.onDestroy()    
  }
}
```



