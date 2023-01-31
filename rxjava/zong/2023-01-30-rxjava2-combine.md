# 合并流
RxJava中Observable就像是一个水流, 持续往外流出数据. 而明显在实际开发过程中, 就能碰到同时有多个流要协同操作的情况. 这时就能应用到本章的内容了. 
这种多个流协作的例子有很多, 比如说: 
* 打开一个页面, 要同时取后台2个endpoint的数据, 都成功后把两个数据组合成一个新数据
* 打开一个页面, 要求按顺序地去访问后台2个endpoint
* 点击按钮就去请求后台endpoint; 但短时间内多次点击按钮, 希望不要发出多个请求, 这时怎么处理每次点击发出的流? 
* ... ...

# 一. 前后相继执行 concat

```kotlin
  Observable.just(111, 222)
      .concatWith(Observable.just(0))
      .subscribe { datum -> println("szw d$datum") }
      .clearBy(disposables)  //=> 111, 222, 0
```

若是前面的流是一直在执行的, 如`Observable.interval`, 那后面的流就一直不能执行的: 

```kotlin
  Observable.interval(1, TimeUnit.SECONDS)
      .concatWith(Observable.just(-1))
      .subscribe { datum -> println("szw d$datum") }
      .clearBy(disposables)  //=> 0, 1, 2, ....
  // (永远不会有-1被打印, 因为ob1总不结束. 只有前一个ob complete了, 才会concat到下一个ob去)
```

## 2. concatMap
其实就是 map转为另一个observable, 然后原来的与新的Observable是concat执行的

```kotlin
  /*
  本来按照时间顺序, 应该是:
  100ms时, ob1发出0; 然后250ms, 发出(0,0); 400ms(0,1)
  200ms时, ob1发出1; 然后350ms, 发出(1,0); 500ms(1,1)
  但因为用了concatMap, 即会concat, 所以实际顺序是要上一个发完了才轮到下一个
  => (0,0), (0,1), (1,0), (1,1)
    */
  val ob1 = Observable.interval(100, TimeUnit.MILLISECONDS).take(2)
  val ob2 = Observable.interval(150, TimeUnit.MILLISECONDS).take(2)
  ob1.concatMap { d1 -> ob2.map { d2 -> "($d1, $d2)" } }
      .subscribe { datum -> println("szw d$datum") }
      .clearBy(disposables)  //=> (0,0), (0,1), (1,0), (1,1)
```

另注: 后面会讲解`flatMap`, 来让你理解其与`concatMap`的区别

## 3. concatAll
RxJS中有`concatAll`操作符, 其实`concatMap = concatAll + map`
不过不理解也没事, 因为RxJava中没有这个操作符. 一般来说concatMap基本就够用了 

