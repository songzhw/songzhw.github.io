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

# 有损背压

## 背压 (BackPressure)

## throttle


## debounce


## sample


## windows
