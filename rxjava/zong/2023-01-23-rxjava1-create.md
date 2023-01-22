# 一. 创建Observable


## 1.1 可能混淆的create与generate

看这两个static操作符, 名字真的好像. 所以这里要特别说明下. 


### 1. create: 创建一个Observable, 自己发next, complete, error


```kotlin
// 1. Observable.create<T> { emitter -> }
var ob: Observable<Int> = Observable.create<Int> { observableEmitter ->
    observableEmitter.onNext(10)
    observableEmitter.onNext(11)
    // observableEmitter.onError(RuntimeException("error1"))
    // observableEmitter.onComplete()
} //=> 10, 11 
```


### 2. generate

generate()的方法, 主要有3个不同的形式: 


#### 1). generate { emitter -> .. }

注意, , generate是若没complete就无休止地发数据

```kotlin
var ob = Observable.generate<Int> { emitter ->
            emitter.onNext(20)
            emitter.onComplete()
        } //=> 20, 20, 20, 20, .... (不停地发送20)

var ob = Observable.generate<Int> { emitter ->
            emitter.onNext(20)
            emitter.onComplete()
        } //=> 20 (因为complete了, 所以不会无何止发送20)

```


#### 2). generate(supplierFn, BiConsumerFn)

这里涉及到的类型: 

```kotlin
public interface Supplier<T> {
    T get() ;
}

public interface BiConsumer<T1, T2> {
    void accept(T1 t1, T2 t2);
}
```


现在我们来看个例子. 这个例子还会利用另一个函数`getCache2()`

```kotlin
fun getCache2() = "hello"

val ob = Observable.generate(
    Supplier { getCache2() },
    BiConsumer { initValue: String, emitter: Emitter<Int> -> 
        emitter.onNext(initValue.length) 
    }
) //=> 5, 5, 5, 5, ... (不停地发下去)
```


和第1)小点一样, 我们不手动complete, 那generate就会一直产生数据.  (而`create`不会!)

所以我们可以这样来表示: 只发送5次

```kotlin
var index2 = 0
ob = Observable.generate(
    Supplier { getCache2() },
    BiConsumer { initValue: String, emitter: Emitter<Int> ->
        emitter.onNext(initValue.length)
        index2++
        if (index2 >= initValue.length) {
            emitter.onComplete()
        }
    }
) //=> 发送了五次5 (因为最后complete了, 所以只会发五次5)
```


#### 3). generate(supplierFn, BiFunctionFn)

这里涉及到的类型: 

```kotlin
public interface BiFunction<T1, T2, R> {
    R apply(T1 t1, T2 t2);
}
```


这个的签名与第2)点的区别是, 第二参从BiConsumer变成了BiFunction, 即有一个返回值了. 那这个BiFunction返回的返回值有什么用呢? 

:  上面的`BiConsumer {initValue, emitter -> }`中的initValue就是 第一参(`Supplier`)的返回值, 不会变的

而这里`BiFunction`的返回值会变为下一次initValue. 是的, 即initValue会变

```kotlin
// 第二参为BiConsumer时只要消费就行, initValue不会变, 就是Supplier的返回值;
// 第二参为BiFunction时, 那initValue会变, 变为BiFunction的返回值 --> 这就类似于一种for(int i = 0; i < 3; i++)的做法了
var index4 = 0
ob = Observable.generate(
    Supplier { getCache2() },
    BiFunction { initValue, emitter ->
        emitter.onNext(initValue.length)
        index4++
        if (index4 >= 4) {
            emitter.onComplete()
        }
        "$initValue$index4"
    }
) 
//=> (initValue = hello) 5, (hello1) 6, (hello12) 7, (hello123) 8
//=>    因为后面complete了, 就不再发数据了

```



#### 4). 注意

generate的lambda中只能onNext一次, 不然会报错: 

```kotlin
// crash: java.lang.IllegalStateException: onNext already called in this generate turn
// 所以generate一次只能onNext一次
ob = Observable.generate(
    Supplier { getCache2() },
    BiFunction { initValue, emitter ->
        emitter.onNext(initValue.length)
        emitter.onNext(-1)  //=> crash here
        "${initValue}x"
    }
)
```


## 1.2 (同步) 发出有限个数据

* `just( vararg T)` : 发送有限几个数据 (备注: RxJS中叫`of`)

* `range(start, count)`: 从start发送count个数据


```kotlin
        var ob = Observable.just(2, 3)  //=> 2, 3
        var ob = Observable.range(2, 3) //=> 2, 3, 4
```
