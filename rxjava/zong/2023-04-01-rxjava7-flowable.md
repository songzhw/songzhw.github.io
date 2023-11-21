# 背压 (Backpressure)
这是一个工程用语, 指上游来的水流或水蒸汽过多了, 下游受不了, 就发生了逆流之内的非正常情况.
放到RxJava世界中也是类似, 就是上游发出来的数据过多了, 搞得下游处理不了这许多, 这种情况就叫背压. 

## 历史: 从RxJava1到RxJava2+
RxJava1的时代, 很多事件不能正确处理背压的情况, 从而会扔出`MissingBackpressureException`. 

比如说下面的RxJava1时代的代码: 
```kotlin
Observable.interval(1, TimeUnit.MILLISECONDS)
  .observeOn(Schedulers.newThread())
  .subscribe { Thread.sleep(1000); print("----> $it");}
```
这样当发出了第16个数据时, 接收方还在sleep, 这就引发了背压
![image](img/image-20231120190432-oqtp738.png)

=> 是的, 在RxJav1时代, 缓存池大小就是16. 超出来就会发生MissingBackpressureException. 

所以到了RxJava2+的时代, 为了更好地处理背压, RxJava引入了多种发生背压时的处理策略:
* 有创建一个cache先存着这些暂时处理不了的数据, 
* 或是处理不了的数据就丢弃这些新来的数据, 
* 或是处理不了时就报错
* ....

为了更好应用这些策略, RxJava破天荒地引入了`Flowable`的概念.
以前RxJava1时代的Observable, 现在成了` = Observable + Flowable`
* `Observable`是不含有背压处理的流
* `Flowable`是含有背压处理的流, 即自带了上面所说的各种处理策略. 

# Flowable 
口说无凭, 我们来看个例子就知道了
```kotlin
val upstream : Flowable<String> = Flowable.create({ FlowableEmitter<String> emitter -> 
  emitter.onNext(...)
}, BackpressureStrategy.ERROR);
```

看到了吧, Flowable的创建会自带背压策略的.  这里的策略可选值有以下几种: 
```kotlin
public enum BackpressureStrategy {
    //不指定背压策略 (那背压时就扔出MissingBackpressureException哦)
    MISSING,
    //出现背压就抛出异常
    ERROR,
    //指定无限大小的缓存池，此时不会出现异常，但无限制大量发送会发生OOM
    BUFFER,
    //如果缓存池满了就丢弃掉之后发出的事件
    DROP,
    //在DROP的基础上，强制将最后一条数据(最latest的一条数据)加入到缓存池中
    LATEST
}
```

## Observable转Flowable
这个RxJava自己就支持了, 它的做法是: 
```kotlin
val aFlowable = aObservable.toFlowable(BackpressureStrategy.xxx)
```


## Flowable的实际使用
因为加了背压处理了, 所以很多三方库都倾向于使用Flowable, 而不是用Observable.
比如说我们的androidx-liveData库中, 就有一个extension方法, 叫`toLiveData()`, 把RxJava的一个流转成LiveData.

但要注意, 这个ext方法是Flowable的, 即:
```kotlin
fun Flowable.toLiveData() {...}

// 但没有 fun Observable.toLiveData() {....} 的方法哦!
```

# Processor
RxJava1中的Subject, 现在也因为背压的关系, 也分成了`= Subject(无背压) + Processor(有背压)`.

所以我们现在有这几种类: 
| 有背压            | 无背压          |
|-------------------|-----------------|
| PublishProcessor  | PublishSubject  |
| BehaviorProcessor | BehaviorSubject |
| ReplayProcessor   | ReplaySubject   |

跟Subject相比, Processor的功能是一样的, 只是多一个背压的功能而已. 

# 备注
RxJS, RxDart, RxSwift中都是没有Flowable这个概念的, 都是一个Observable概念的. 

我个人其实更喜欢这种, 就是一套代码, 现在RxJava2+中因为Observable与Flowable分立, 结果我们有时候要处理两种情况, 如: 
```kotlin
fun Obserable.align() = ...
fun Flowable.align() = ....
```
造成了大量重复代码. 

# 背压的操作符
前面讲过, RxJS, RxSwift这些都是一套代码, 一个observable管全部, 不用再分成obserable与flowable. 那这些RxXX是如何处理背压的, 它们的处理方式也同样适用于RxJava吗? 

: 答案, 是的, Rx世界中一些公用的操作符, 也可以用来处理背压.  举个例子, 当有大量数据时, 你可以每1秒取一次数据, 让下游慢慢消化这个数据, 这样就不会有背压了. 这些操作符通行于RxJava, RxJS, RxDart, ..., 而且也能用于上游数据很多时. 下面就来一个个地看

## 有损背压的几个操作符
有损背压有throttle, debounce, sample方式.
这几种都在[rxjava 03: filter](https://github.com/songzhw/songzhw.github.io/blob/master/rxjava/zong/2023-02-02-rxjava3-filter.md)这一文章中有过介绍了. 这里就不赘述了.

这种方式全是有损的, 即有过多数据过来时, 我就只隔着一个时间段再去取. 这中间时间里发出的数据就扔了, 即"有损"的. 

## 无损背压的几个操作符
想要无损, 那就要把所有数据都缓存起来. 

我们可以用
* `buffer(count)` 来缓存多少个数据
* 也可以把observable转为`flowable`, 并指明buffer策略
```kotlin
  Observable.interval(1, TimeUnit.MICROSECONDS)
      .buffer(1000)

  Observable.interval(1, TimeUnit.MICROSECONDS)
      .toFlowable(BackpressureStrategy.BUFFER)
```

# 总结
背压(backpressure)是一种上游数据过多, 超出下游处理能力的情况. 这种情况下, 我们多用Flowable或Processor来做为流.  原因是它们自带了BackpressureStrategy, 方便我们指明要如何处理上游数据过多的情况. 

