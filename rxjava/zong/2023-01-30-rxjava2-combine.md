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



# 二. 先来先得, 人人有份: Merge

```kotlin
  // 先到先得, 所以是A0(100ms), B0(150ms), A1(200ms), B1(300ms)
    val ob1 = Observable.interval(100, TimeUnit.MILLISECONDS).take(2).map { x -> "A$x" }
    val ob2 = Observable.interval(150, TimeUnit.MILLISECONDS).take(2).map { x -> "B$x" }

    // ob1.mergeWith(ob2) // 方式一
    Observable.merge(ob1, ob2)
        .subscribe { datum -> println("szw $datum") }
        .clearBy(disposables)  //=> A0, B0, A1, B1
``` 

当然, 你要是使用RxJava中独有的 `Observable.mergeArray(ob1, ob2)`, 也是和上面一模一样的效果的. 

## 2. merge with maxCurrency
当你有5个流要同时执行并使用merge时, 你因为一些特殊需求, 要求最多同时2个在执行. 只要一个完成了, 后面候补的就递进来执行. 这其实就是在调用: 
`Observable.mergeArray(maxCurrency = 2, bufferSize = 1, ob1, ob2, ob3, ob4, ob5)` 

```kotlin
  val ob1 = Observable.interval(100, TimeUnit.MILLISECONDS).take(2).map { x -> "A$x" }
  val ob2 = Observable.interval(150, TimeUnit.MILLISECONDS).take(2).map { x -> "B$x" }
  val ob3 = Observable.interval(60, TimeUnit.MILLISECONDS).take(2).map { x -> "C$x" }

  // maxConcurrency = 2, bufferSize = 1, 说明最多同时进行2个 (所以C0后面才出来)
  Observable.mergeArray(2, 1, ob1, ob2, ob3)
      .subscribe { datum -> println("szw $datum") }
      .clearBy(disposables)
  //=>(100ms) A0, (150ms) B0, (200ms) A1    [现在ob1完结了, 空出来位置了, ob3可以进场了]
  //=> (260ms) C0, (300ms)B1, (320ms) C1    [这就证明了同时最多进行maxConcurrency个ob]

```

备注: 若你熟悉RxJS, 其实RxJS的merge也有这个maxCurrency参数, 不过RxJS中是叫`concurrent`: 

```js
const merged$ = ob1.merge(ob2, ob3, 2) //这个2就是concurrent
```

### bufferSize这个参数没什么用
p.s. 经过实践, 发现第二参bufferSize没什么用. 它设置什么值(只要是正数), 都不影响最后的结果

```kotlin
    val ob1 = Observable.interval(100, TimeUnit.MILLISECONDS).take(2).map { x -> "A$x" }
    val ob2 = Observable.interval(150, TimeUnit.MILLISECONDS).take(2).map { x -> "B$x" }
    val ob3 = Observable.interval(60, TimeUnit.MILLISECONDS).take(2).map { x -> "C$x" }
    val ob4 = Observable.interval(40, TimeUnit.MILLISECONDS).take(2).map { x -> "D$x" }
    val ob5 = Observable.interval(30, TimeUnit.MILLISECONDS).take(2).map { x -> "E$x" }

    // maxConcurrency = 2, bufferSize = 1, 说明最多同时进行2个 (所以C0后面才出来)
    Observable.mergeArray(2, 1, ob1, ob2, ob3, ob4, ob5)
        .subscribe { datum -> println("szw $datum") }
        .clearBy(disposables)
    //=>A0, B0, A1, C0, (300ms)B1, (320ms)C1, (340ms)D0, (350ms)E0, (380ms) E1, (380ms)D1
    //=> [所以bufferSize不够, 也不会crash的, 放心 -- 也就是从实践中感觉 , bufferSize好像没什么用]

```

但是注意, maxCouncurrency与bufeferSize一定得是 > 0的正数, 等于0都不行(会crash, 说IllegalStateException)


### 备注

备注: `merge`方法没maxCurrency这个参数, 但`mergeArray`有. 

备注: 若是 maxCurrency = 1, 那其实就成了`concat`的效果了. 

备注: 后面要讲的flatMap也可以有maxCurrency这个参数哦!

## 3. flatMap
flatMap这个名字其实有点让人误解. 但是回顾下RxJS的历史, 能帮我们更加了解它. 
在RxJS v4版本的阶段, merge流后, 每个数据再map成一个新的流, 这种操作就是叫`mergeMap`
但是在RxJS v5版本及以后, `mergeMap`改名成了`flatMap`.
-> 这样一来, 我们就知道, 其实flatMap与merge的关系,  就像是concatMap与concat的关系是一样的. 

正如上面所说, flatMap其实是merge的操作, 所以它其实也是`先到先用, 人人有份`的特点. 这样下面的代码就能理解了: 

```kotlin
  /*
  本来按照时间顺序, 应该是:
  100ms时, ob1发出0; 然后250ms, 发出(0,0); 400ms(0,1)
  200ms时, ob1发出1; 然后350ms, 发出(1,0); 500ms(1,1)
  flatMap本身就是mergeMap, 所以是merge的(先到先用)的时间顺序
  => (0,0), (1,0), (0,1), (1,1)
  */
  val ob1 = Observable.interval(100, TimeUnit.MILLISECONDS).take(2)
  val ob2 = Observable.interval(150, TimeUnit.MILLISECONDS).take(2)
  ob1.flatMap { d1 -> ob2.map { d2 -> "($d1, $d2)" } }
      .subscribe { datum -> println("szw d$datum") }
      .clearBy(disposables)  //=> (0,0), (1,0), (0,1), (1,1)
```

备注: 若是flatMap改为concatMap, 则结果会是`(0,0), (0,1), (1,0), (1,1)`, 即一个流一个流地来, 而不是先到先用. 


### flatMap也有maxCurrency参数

```kotlin
btnSingle.clicks()
    .doOnNext{ println("szw click")}
    .flatMap(
        {Observable.interval(500, TimeUnit.MILLISECONDS)},
        1) // maxConcurrency = 1
    .subscribe { println("szw got : $it") }
```
这时我就是点击按钮多次, 也只会有第一个ob在运行 (因为interval是无限的, 所以它会总只有第一个ob在运行)

#### 流过多时, 多余出来的流会被如何处理
那问题来了, 要是第一个ob还在执行, 来了第二个, 第三个ob, 这时怎么办? 后来的两个Ob会被缓存下来, 还是会被丢弃?
: 答案是缓存下来. 第一个Ob完了, flatmap就从缓存中取出第二个Ob来执行

```kotlin
btnSingle.clicks()
    .doOnNext { println("szw click") }
    .flatMap(
        {
            Observable
                .interval(500, TimeUnit.MILLISECONDS)
                .take(4)
        },
        1
    )
    .subscribe { println("szw got : $it") }
```    
下面我点击三次, 注意"szw click"出现的位置, 日志依次是:
szw click, 0, 1, szw click, 2, szw click, 3
0, 1, 2, 3
0, 1, 2, 3

=> 备注: 其实flatMap(fn, 1)就是concatMap的效果啦

备注: rxswift中, switchMap没有, 但其实就是 flatMapLatest()
rxswift中, exhaustMap也没有, 但其实就是 flatMapFirst()