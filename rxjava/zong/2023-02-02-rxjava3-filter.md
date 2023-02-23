这一章主要是讲过滤操作符, 包括 filter, first, last, take, skip, distinct, elementAt等. 
但是暂不包括debounce, throttle, sample, window这些涉及到backpressure的操作符. (这些请见后面的文章)

## first, last
`filter`是最常见的操作符这里就不多做介绍了. 这一小节主要是介绍取前N个, 取最后N个的first与last一系列操作符. 

### first一系
我们经常能看到`first(defaultItem)`以及`firstElement()`两个first系的操作符. 它们的区别是:
* `first(defaultItem)`, 若是流中没数据就使用defaultItem. 其返回的是是个Single<T>, 即只有succ(value)与error两种结果. 
* `firstElement()`是返回一个Maybe. 即要么发送0个数据, 直接complete; 要么发送一个数据 succ(T); 要么就是error出错. 

```kotlin
  Observable.just(3, 1, 6, 5, 8)
      .firstElement() //=> 3  //返回Maybe<T> 

  Observable.just(3, 1, 6, 5, 8)
      .first(-1) //=> 3 //返回Single<T>, 没数据就用参数(defaultItem)-1 

  // 前3个数据
  Observable.just(3, 1, 6, 5, 8)
      .take(3) //=> 3,1,6       
```

### last一系
对应着first取开头的数据, 我们还有一组last操作符, 来取最末的几个数据:

```kotlin
  Observable.just(3, 1, 6, 5, 8)
      .last(-1) //=> 8 //返回Single<T>, 没数据就用参数(defaultItem)
  Observable.just(3, 1, 6, 5, 8)
      .takeLast(3) //=> 6, 5, 8     
  Observable.just(3, 1)
      .takeLast(3) //=> 3, 1 (不足3个也不会报错哦)                           
```

备注, 只有last(defaultItem), 没有lastElement()操作符哦!

## take一族

主要是分为take多个元素, 或是take某一时间段内的所有元素. 这些都较简单, 就不详细解释了, 看下面的代码就已经很明白了

```kotlin
// take(n), takeLast(n)
    Observable.just(3, 1, 6, 5, 8).take(3)//=> 3, 1, 6
    Observable.just(3, 1, 6, 5, 8).takeLast(3)//=> 6, 5, 8

// take(time, timeUnit), talkeLast(time, timeUnit)
    Observable.interval(80, TimeUnit.MILLISECONDS)
        .take(200, TimeUnit.MILLISECONDS)
        .subscribe { datum -> println("szw got $datum") } //=> 0,1 (80ms, 160ms)

    Observable.interval(80, TimeUnit.MILLISECONDS)
        .take(10)
        .takeLast(200, TimeUnit.MILLISECONDS)
        .subscribe { datum -> println("szw got $datum") } //=> 7, 8, 9 (640ms, 720ms, 800ms)              

// take没这签名, 但takeLast有, 即: takeLast(count, time, timeUnit)
// 这时就是若窗口中出M个, 而第一参为count, 那最终就是取 `min(M, count)`个最后的元素
    Observable.interval(80, TimeUnit.MILLISECONDS)
        .take(10)
        .takeLast(2, 200, TimeUnit.MILLISECONDS) //若没第一参, 那就是产出7,8,9一共三个元素
        .subscribe { datum -> println("szw got $datum") } //=> 8, 9


```

注意1: takeLast(count, time, unit)是有的. 但没有take(count, time, unit)这个方法哦!


## 其它的一些过滤
也都是比较简单的操作符, 就只看例子就明白了

### distinct一系
`distinct`是在整个流中去除
`distinctUntilChanged`则只是去除连续的重复数据而已.

```kotlin
    Observable.just(1, 2, 3, 3, 2, 4, 1)
        .distinct()//=> 1, 2, 3, 4

    Observable.just(1, 2, 3, 3, 2, 4, 1)
        .distinctUntilChanged()//=> 1,2,3,2,4,1 (只剔除连续的重复数据)          
                
```

### 其它的一些过滤符

```kotlin
 // ignoreElements不关心数据, 只关心完成或出错
    Observable.just(1, 2, 3, 3, 2, 4, 1)
        .ignoreElements() //返回一个Completable


    Observable.just(1, 4, 7)
        .elementAt(2) //返回个Maybe. 参数Index是从0开始计算的.
        .subscribe { println("szw(34) $it") } //=> 7

    Observable.just(1, 4, 7)
        .elementAt(3, -1) //返回个Single. index是从0开始计算的. 第二参是defaultItem
        .subscribe { v -> println("szw(35) $v") } //=> -1                
```


--------

# 背压 (BackPressure)
上游在短时间内发出大量数据, 多到下游接收不了了. 这个现象就叫背压.  那处理背压的策略主要有两种:
1). 缓存起来: 若太多了下游接收不了了, 这时策略之一就是缓存起来. 这样等什么时候数据不多时, 我再从缓存中取出来, 保证我下游总能正常地处理  -- 当然, 若一直不停地发, 缓存空间不够了, 那还会造成问题的. 

2). 丢弃: 相当于下游搞了一个阀门, 只接收有限的数据. 多余的数据就给丢弃.  这种有所丢失的策略就叫有损背压. 这也是我们这一部分的重点 

## 有损背压
有损背压有throttle, debounce, sample方式. 每种方式的不同仅在于"什么时候开阀门"而已. 下面我们一一来讲解.

## debounce

### debounce(time, timeUnit)
若我们有一个按钮, 点击后去请求后台或取数据库数据(总之就是耗时的操作). 我们自然不然用户短时间内多次点击它. 这时我们可以用: 
`debounce(1, TimeUnit.SECOND)`.
这样的话, 你怎么点击按钮都不会触发那耗时操作的. 
只有, 只有当你停止点击按钮后, 一秒后就会触发这个耗时操作.

所以, `doubnece(1s)`其实就是说: `你总发来数据, 我都不处理的; 但是你一旦停了发数据达1s之久, 那我就处理了`. 


### debounce(ob2)
这时的参数是另一个Observable (ob2). 

#### 例子1: 当我点击了另一个按钮, 才触发操作
```kotlin
// 点击btn23多次也没用, 就是走不到subscribe这里去
// 直到你点击了btn24, 马上就走到了subscribe里了
// 所以debounce(debounceIndicator)就是说参数indicator发出数据了, 这时就不drop了开始接收.
var num23 = 1
btn23.text = "debounce indicator2"
btn24.text = "tirgger prev button's debounce!"
btn23.clicks()
    .doOnNext { println("szw clicks") }
    .map { ++num23 }
    .debounce { intNumber ->
        btn24.clicks()
    }
    .subscribe { println("szw consume clicking $it") }
```        

#### 例子2
所以对于 `ob1.debounce(2s)`, 我们可以用这个重载函数达到一样的效果, 即

```kotlin
ob1.debounce{_ -> Observable.timer(2, TimeUnit.SECONDS)}; // 效果等同于 debounce(2, TimeUnit.SECONDS);

```

## sample


## throttle




