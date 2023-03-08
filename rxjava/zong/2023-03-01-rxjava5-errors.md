# 五. 错误处理

本章开讲之前, 先讲一个本章中总要用到的出错函数. 我们就放到流里来故意引发错误, 这样我们就方便对错误进行处理了. 

```kotlin
fun jinx(number: Long): Long {
    if (number == 4L) {
        throw RuntimeException("4不吉利")
    }
    return number
}
```


Rx世界中, 对错误的处理一般分三种:

* 出错后, 什么都不做, 让它crash掉整个应用
* 出错后, catch住error, 不要crash
* 出错后, 重试几次, 若还不行再扔出错误去



## 一般的错误处理


### 下游处理error

RxJava入门级的教材(网上有很多), 都会讲, 可以在subscribe中使用onError来处理错误. 

```kotlin
stream.subscribe(
  {datum -> ... }, /* handle result */
  {error -> ....}, /* handle error */
  { ... }, /* handle completed event */
)
```

若是stream中出错了, 而我们只有处理result的lambda, 没有处理error的lambda, 那整个应用就会crash. 


### 监听错误 

`doOnError{ error -> ...}` 就是监听错误. 

若有错误发生时, 加上这个操作符并不能处理掉错误. 这个操作符只是做一些监听.如: 

```kotlin
backendRequestStream
    .doOnError { err -> /* 对error情况做埋点 */ }
    .subscribe(
      {datum ->  /* 刷新页面 */ },
      {error ->  /* 展示error dialog */ },
    )
```


当然, 你要是想写成: 

```kotlin
backendRequestStream
    .subscribe(
      {datum ->  /* 刷新页面 */ },
      {error ->  
        /* 对error情况做埋点 */
        /* 展示error dialog */ 
      },
    )
```

也是可以的, 就是看起来有点臃肿.  在Rx世界中, 最起码是我个人感觉, 代码要优雅. 最好每个操作符的处理都不太长, 这样整个代码显得一般匀称, 而不是头重脚轻, 或头轻脚重.  所以我个人感觉用监听的写法更好些.


#### 注意操作符的顺序

`doOnError`也得注意顺序:

```kotlin
Observable.interval(100, TimeUnit.MILLISECONDS)
    .take(6)
    .doOnError { err -> println("szw detect $err") }
    .map(::jinx)  // error发生在这一步
```

这样就不会有错误被监听到. 因为jinx函数是发生在doOnError之后的. 


要想监听成功, 正确的做法是: 

```kotlin
Observable.interval(100, TimeUnit.MILLISECONDS)
    .take(6)
    .map(::jinx)  // error发生在这一步
    .doOnError { err -> println("szw detect $err") }
```


## catch住错误 

在RxJava中处理错误是用`catch`, 

但在Java, Swift等语言中, catch多是做为`try-catch`的一部分而成为了关键字, 所以不能被用来命名函数/操作符. 这时RxSwift中这个操作符就是改名为了`catchError`. 而在RxJava中(其实RxDart也一样), 这个操作符被细分成了: 

* onErrorReturn {err -> T}

  * 若出现错误, ob1就不进行下去了. 直接发送一个T出来, 做为最后一个数据
* onErrorReturnItem(T)

  * 若出现错误, ob1就不进行下去了. 直接发送一个T出来, 做为最后一个数据 (这是上面的一个语法糖)
* onErrorResumeNext{err -> ob2 }

  * 若出现错误, ob1就不进行下去了. 下游转为监听ob2的数据了
* onErrorResumeWith{errorStream -> it.onNext(T) }

  * 若出现错误, ob1就不进行下去了.  这时it参数可以发出onNext(T), onError(..), onComplete(). 具体做什么可以自己定
* onErrorComplete()

  * 若出现错误, ob1就不进行下去了. 直接complete.


所以, 若上游是这样一个流

```kotlin
Observable.interval(100, TimeUnit.MILLISECONDS)
    .take(6)
    .map(::jinx) //因为是top-level函数, 所以不用"obj::fun"中的obj都行!
```


那一一加上上面的操作符, 结果分别是: 

```kotlin
// return系 (ob1结束)
  .onErrorReturn { throwable -> -1 } 
  .subscribe(::println)//=> 0,1,2,3,-1

  .onErrorReturnItem(-100)
  .subscribe(::println)//=> 0,1,2,3,-100

// resume系 (ob1结束, 下游转为监听新的ob2)
  .onErrorResumeNext { throwable -> Observable.just(-200, -201) } 
  .subscribe(::println)//=> 0,1,2,3,-200, -201

  // 这里可以用it.onError(..), it.onComplete(), 你自己决定 
  .onErrorResumeWith { it.onNext(-555) } 
  .subscribe(::println)//=> 0,1,2,3,-555

// complete系
  .onErrorComplete()
  .subscribe(::println)//=> 0,1,2,3
```




## 重试

### retry几次

比如说retry(3)表示重试三次, 还不行就扔出error

注意, 这里加上原来执行的一次, 还有三次重试的, 所以一共会执行四次. 


```kotlin
Observable.interval(100, TimeUnit.MILLISECONDS)
    .take(6)
    .map { it + 2 } //快一点jinx, 方便我统计. 即会产出2,3,4,5,6,7
    .map(::jinx) //因为是top-level函数, 所以不用"obj::fun"中的obj都行!
    .retry(2)
    .prints(disposables)
//=>2,3; 2,3(retry第一次); 2,3(retry第二次); crash(retry完后仍出错, 所以会crash)

```


这其实等同于另一种写法: 

```kotlin
  ...
  .map(::jinx)
  .retry{retryCount, throwable -> retryCount <= 2} //即等于retry(2)
```

这里的参数retryCount是个整数, 表示重试了多少次了. 它是从1开始计数的


### retry until stopPredict

```kotlin
var index = 0

...
  .retryUntil { index++; index > 2 } //满足条件就stop retry了. 这个lambda是: ()->Boolean 类型
```

所以这也是相当于retry(2)了



### retry when ...

当我们使用`retryWhen{ob2}`时, 在出错后, 会进入到retryWhen里去.

重试若还有错, 还会进入到retryWhen里. 如此循环....


```kotlin
    .map(::jinx) //因为是ittop-level函数, 所以不用"obj::fun"中的obj都行!
    .retryWhen { ob -> ob.delay(3, TimeUnit.SECONDS) }
    .prints(disposables) //=> 每3秒重试一下 (注意, 并不是只重试一次哦!!!)
```

所以这种代码, 就相当于是每3秒重试一次. 若一直有错误就一直重试, 完全没有终止. 


只是要注意一点, retryWhen{ob -> ... } 一定要利用这个ob. 若是下面的代码, 是不会有3秒后重试的效果的: 

```kotlin
    .map(::jinx) //因为是ittop-level函数, 所以不用"obj::fun"中的obj都行!
    .retryWhen { Observable.timer(3, TimeUnit.SECONDS) }
    .prints(disposables) //=> 每3秒重试一下 (注意, 并不是只重试一次哦!!!)
```


### 案例: 重试3次, 每次重试的间隔越来越长

上面的retryWhen{ob -> ob.delay()}就能保证让重试之间有些时间间隔, 而不是立即去重试. 

若我们还想更复杂些, 如最多重试3次, 每次出错后应该间隔个2秒, 5秒, 8秒再重试, 这时要怎么办?

: 那我们就要利用retryWhen来保证有间隔, 同时自己记录重试次数.  根据这一思路, 我们写成这样子 :

```kotlin
fun <T> Observable<T>.retryWithDelay(
    maxRetries: Int,
    firstDelayInMs: Long,
    retryIntervalInMs: Long
): Observable<T> {
    var retryCount = 0
    var delayTime = firstDelayInMs

    return retryWhen { errorOb -> //这里一定要使用这个errorOb, 不然没有retry的效果!!!
        errorOb.flatMap { throwable ->
            if (retryCount < maxRetries) {
                val scheduleOb = Observable.timer(delayTime, TimeUnit.MILLISECONDS)
                delayTime += retryIntervalInMs
                retryCount++
                scheduleOb
            } else {
                // Observable.just(throwable) //这就相当于总是在重试了. (上下两次重试之间没有间隔了)
                Observable.error(throwable) //这会crash掉app. 但这正是retry有限次的意义嘛
            }
        }
    }
}
```

注意, 因为是在使用`retryWhen`, 所以lambda的参数`errorOb`一定要利用, 这才有了上面使用`errorOb.flatMap { Observable.timer()}`的用法 .