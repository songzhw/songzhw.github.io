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

