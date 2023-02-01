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


### 实例
如点击某一按钮, 与拖动按钮都要启动一个事件(比如说startAnimation), 那就可以用: 

```kotlin
Observable.merge(clickStream, touchStream)
  .subscribe { startAnimation() }
```

又比如说用户在进行一个步骤很多的过程, 为了方便用户有事退出, 我们要每30秒记录下用户进行到哪一步了. 这样下次用户再重新打开这个过程, 我们就会直接进行到上一次保存的地方. 
同时, 只要用户每进一步, 即我们自己记录的progress有变化, 我们也记录下用户进行到哪一步了.
即两个流的下游都是一样的操作, 这时我们就可以: 

```kotlin
val progressStream = Progress.stream()
val every30Sec = Flowable.interval(30, TimeUnit.SECONDS)
    .map { time -> Progress.getProgresss()}

Flowable.merge(progressStream, every30Sec)
  .subscribe { saveProgress(it) }

fun saveProgress(progress: Progress) { ... }  
```

备注: 这个例子2, 其实有一个细节点. 那就是progressStream是一个Flowable<Progress>, 但是every30Sec是一个Flowable<Long>.  而我们的merge是要求上游都是同样的Flowable<T>, 即数据都是T. 若有一个上游的数据类型不一样, 那就会报错, 编译不了了. 所以我们才在上面使用了map, 来让interval产生的Long类型转化为Progress类型. 

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


### flatMap的应用场景 
比如打开某一页时, 这个页面要请求2个后台接口才能开始渲染. 但这两个后台接口有依赖关系, 第一个接口要得到userId, 第二个接口使用这个userId去得到userInfo. 这时就适合使用flatMap, 即拿到userId后, 去转化为另一个流, 就可以使用到flatMap

```kotlin
userApi.getUserId() 
  .map {resp1 -> resp1.userId}
  .flatMap {userId -> userApi.getUserInfo(userId)}
  .map {resp2 -> resp2.jsonToUser()}
  .subscribe { userInfo-> renderPage(userInfo)}
```

## 4. zip
上面的`concat`, `merge`都有一个共同特点, 就是它们的上游的数据都会流到下游去. 比如上游1发出A与B, 而上游2发出C. 那下游一共就会收到三个数据, 被调用三次, 分别是收到A, B, C

那要是所有上游的数据都只提供一部分数据, 最后要组合后给下游 (即下游只收到一次数据). 应用场景就是多线程下载某一文件, 我们分成10份, 每份下载文件的1/10, 每部分都是独立的流. 当然每一个流下载完了自己的部分, 并不能直接给用户. 而应该是要等到所有10份全下载完了, 组装起来才给用户用. 是的, 用户并没有收到10次数据, 只是收到一个组装后的完整数据. 这种情形下, 使用`zip`才是合适的. 

例子: 

```kotlin
  // zip的任一上游complete时, zip自己就也complete了. 所以结果只看到2对!
  val ob1 = Observable.interval(100, TimeUnit.MILLISECONDS).take(2).map { x -> "A$x" }
  val ob2 = Observable.just(3, 9, 13, 15)
  ob1.zipWith(ob2) { d1, d2 -> "($d1, $d2)" }
      .subscribe { datum -> println("szw $datum") }
      .clearBy(disposables) //=> (A0, 3), (A1, 9)
```
备注: 从这个事例中也能看到, 只要任一个上游完结了, zip也会完结. 

### zip的上游是同时工作的吗? 
仍是上面10个小组同时下载同一文件的例子. 那要是每个部分都耗时30秒, 那下游收到数据是30秒之后, 还是300秒(30 * 10)之后呢? 
: 先给出答案哦, 是30秒. 这样好, 用户就不用等待这么久了. --> 要是300秒, 那其实是concat的耗时啦, 因为concat要等上一个上游完成, 才能开始下一个.  zip则是所有上游都同时里德. 

```kotlin
  println("szw start the zip")
  val ob1 = Observable.timer(3, TimeUnit.SECONDS)
  val ob2 = Observable.just("hi").delay(2, TimeUnit.SECONDS)
  
  // 问题是, 耗时了5秒, 还是耗时了3秒?
  Observable.zip(ob1, ob2) { d1, d2 -> "($d1, $d2)" }
      .subscribe { datum -> println("szw $datum") }
      .clearBy(disposables)
  //=> 2023-01-25 16:34:11.429: szw start the zip
  //=> 2023-01-25 16:34:14.460 : szw (0, hi)
  // : 答案是3秒. 所以zip的上游是同时进行的. 这就对于一些进入页面要取多个后台数据的endpoint是有好处的, 更快能拿到结果
        
```

## 5. combineLatest
典型的适用combineLatest的场景就是: 气象站.  
假设我们有一个气象站, 它会发出(风力, 温度, 风向)的组合数据给天气预报栏目组. 注意, 只要风力, 温度, 风向这三个(上游)有一个数据变化了, 就要发给天气预报组最新的(风力, 温度, 风向)数据.  --> 这种场景就是: 任一上游有变化了, 就要把所有上游的数据给组合起来, 并发给下游

```kotlin
  val ob1 = Observable.interval(5, TimeUnit.MILLISECONDS).take(3).map { x -> "风力$x" }
  val ob2 = Observable.interval(7, TimeUnit.MILLISECONDS).take(2).map { x -> "$x°C" }
  Observable.combineLatest(ob1, ob2) { w, t -> "($w, $t)" }
      .subscribe(
          { datum -> println("szw $datum") },
          { err -> println("szw err = $err") },
          { println("complete") }
      ).clearBy(disposables)
  //=> (风力0, 0°C), (风力1, 0°C), (风力1, 1°C), (风力2, 1°C), complete
       
```     

### 请求2个后台接口的场景 
若你进入一个页面, 要请求2个后台接口, 然后把这两个response给组合下给用户. 即所有上游都只会发一次数据. 这时你使用 zip, 或是使用 combineLatest, 都是可以的. 这二者的特点就是都会把所有上游的数据给组合起来再发给下游. 

```kotlin
Observable.zip( userApi.getUser(), accountApi.getAccount) // Okay

Observable.combineLatest( userApi.getUser(), accountApi.getAccount) // Okay

```

## 6. 节奏$.withLatestFrom(数据$)
这里的上游有2个, 一个是节奏, 一个是数据. 它们也会把上游数据组合一下再给下游. 但何时发就要看`节奏$`这个流何时发出数据了. 

举个例子, 我们有一个在线手绘的网站. 我们还有一个设置画笔颜色, 画笔宽度的Slider (Android里叫Seekbar). 
明显我们调节Slider时, 只是在设置数据, 并不需要画画
但是当我们鼠标开始绘画时, 就要去读取画笔的信息. 
若下游就是收到数据后进行绘制, 鼠标流提供坐标, 两个Slider的流提供画笔的信息. 那明显是鼠标动了, 才会发数据给下游(即要求下游绘制). 这种场景就适合withLatestFrom

```kotlin
  val mouseMove_ = Observable.interval(1, TimeUnit.SECONDS).take(3).map { x -> "坐标$x" }
  val color_ = Observable.interval(100, TimeUnit.MILLISECONDS).take(100).map { x -> "color$x" }
  val width_ = Observable.interval(150, TimeUnit.MILLISECONDS).take(100).map { x -> "width$x" }
  mouseMove_.withLatestFrom(
      color_,
      width_
  ) { pos, color, width -> "($pos, $color, $width)" }
      .subscribe { datum -> println("szw paint $datum") }
      .clearBy(disposables)
  //=> 日志如下. 这样子就能看出 `节奏$`.withLatestFrom(`数据$`) 的意思所在了
  /*
  2023-01-26 23:27:22.966: szw clicked
  2023-01-26 23:27:23.968: szw paint (坐标0, color9, width5)
  2023-01-26 23:27:24.969: szw paint (坐标1, color19, width12)
  2023-01-26 23:27:25.972: szw paint (坐标2, color29, width18)
    */
```

### withLatestFrom, combineLatest的区别
若是上面的例子是用combineLatest, 那当我们只是去调节paint color这个slider, 也会去绘制, 这就有点奇怪了. 应该是有鼠标动作了才开始画画啊


## 7. race
即多个上游都同时开始工作, 哪个上游先提供了数据, 那下游就认准了这个上游, 一直只接受这个上游的数据.
RxJS中, 这个操作符是叫`race`. 而在RxJava中, 这个操作符叫 `amb`, 即"ambiguous(模棱两可的)"的缩写. 

```kotlin
  // 这个例子表示ob2胜出了, 会一直输出ob2的数据
  val ob1 = Observable.interval(2, TimeUnit.SECONDS).map{x -> "A$x"}
  val ob2 = Observable.interval(1, TimeUnit.SECONDS).map{x -> "B$x"}
  Observable.ambArray(ob1, ob2)
      .subscribe { datum -> println("szw $datum") }
      .clearBy(disposables) //=> B0, B1, B2, B3, B4, ....
```

## 8. switchMap
这是说若持续有流过来, 但上一个流还没做完, 又来了新流. 这时switchMap就会取消掉这个正在做的流, 马上执行新流. 是的, "喜新厌旧"就是switchMap.

使用场景就是常见的点击按钮就去做一个耗时的操作(如去数据库读取, 或是去后台接口读取数据). 但有时用户手误, 或是心急, 一下子点了好几次按钮, 这里要不做下处理, 那就会是执行多次流的请求. 这明显是多余的. 
```kotlin
btnRequestBackend.clicks // RxBinding
  .flatMap { userApi.getUser() }
  .map { resp -> resp.body?.string()}
  .subscribe{ user -> renderPage(user) }
```

要是在这种场景下,我们用switchMap来处理, 那就会是点击多次也没用, 只要还在执行上一个流, 来了新流, 那新流就会生效, 正在执行的旧流就会停止. 这就保证了不会有多次请求.
```kotlin
btnRequestBackend.clicks // RxBinding
  .flatMap { userApi.getUser() }
  .map { resp -> resp.body?.string()}
  .subscribe{ user -> renderPage(user) }
```

另一个更加说明switchMap工作情况的例子: 
```kotlin
  var index1 = 0
  btnSwitch.clicks()
      .doOnNext { index1++ }
      .switchMap { _ ->  Observable.interval(2, TimeUnit.SECONDS).map{y -> "(第${index1}次点击, 数据=$y)"}.take(4)}
      .subscribe { datum -> println("szw $datum") }
      .clearBy(disposables)
  //=> 点击一次 (第1次点击, 数据0), (第1次点击, 数据1), (第1次点击, 数据2)
  //=> 再点一下 (第2次点击, 数据0), (第1次点击, 数据1)
  //=> 再点一下 (第3次点击, 数据0), (第3次点击, 数据1), ...
```

## 9. exhaustMap
RxJS中有`exhaustMap`. 仍以上面的多次点击一按钮为例, switchMap是总是只会执行最新的流. 而exhaust的处理方式正好相反:  *只要前一个流没完成, 新来的流就会被取消掉* . 典型的"干一行, 爱一行", 不贪心多余的. 

可惜不过RxJava中没有这个操作符. 好在我们也可以轻松得利用已有操作符达到这个效果. 其实就是做成一个Flowable, 然后把backpressure strategy设定为DROP, 即上一个没做完, 新来的就会被丢弃. 
同时为了说同时只能执行一个, 我们还得使用`flatMap(maxConcurrency=1, fn)`这个带maxCoucurreny参数的flatmap:
```kotlin
  var index4 = 0
  btnExhaust.clicks()
      .doOnNext { index4++ }
      .toFlowable(BackpressureStrategy.DROP)
      .flatMap(
          { _ ->  Flowable.interval(2, TimeUnit.SECONDS).map{ y -> "(第${index4}次点击, 数据=$y)"}.take(4)},
          1)
      .subscribe { datum -> println("szw $datum") }
      .clearBy(disposables) //=> 效果和concatMap一样
  //=> 点击1次btn, (第1次点击, 数据=0), (第1次点击, 数据=1)
  //=> 这时再点7下button, (第8次点击, 数据=2), (第8次点击, 数据=3) --> 数据没从0开始, 所以还是第一个ob在执行
  //=> 现在流停了.  我们再点击button, 这时数据又开始产生了, (第9次点击, 数据=0), (第9次点击, 数据=1), (第9次点击, 数据=2), (第9次点击, 数据=3)
```


## 10. exhaustMap与concatMap
备注: exhaustMap与concatMap.  concatMap是前一个流还没完成, 新来的流不会被取消, 而是缓存起来, 后面再执行而已.

```kotlin
  var index2 = 0
  btnConcat6.clicks()
      .doOnNext { index2++ }
      .concatMap { _ ->  Observable.interval(2, TimeUnit.SECONDS).map{y -> "(第${index2}次点击, 数据=$y)"}.take(4)}
      .subscribe { datum -> println("szw $datum") }
      .clearBy(disposables)
  //=> 新的点击来了, 也会等上一个完了, 再执行新的点击ob
  //=> 点击btn, (第1次点击, 数据=0), (第1次点击, 数据=1), (第1次点击, 数据=2)
  //=> 再点击btn, (第2次点击, 数据=3), (第2次点击, 数据=0), (第2次点击, 数据=1), (第2次点击, 数据=2), (第2次点击, 数据=3)

```