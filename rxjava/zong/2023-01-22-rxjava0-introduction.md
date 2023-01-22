2023年大年初一, 开始做一个RxJava的总结版本. 这是第一章, 主要讲如何创建Observable的. 

## java与kotlin
整个系列有一个地方要注意的就是, 网上很多资料是java版本, 但是现在kotlin才是主流. 有相当多代码, 要把Java转成kotlin可没这么想当然的容易. 比如说

```java
    Observable<Integer> numbers = Observable.generate(
            this::getCache,
            (prev, emit) -> { emit.onNext(prev * 2); }
    );
```

但是我们简单地复制成kotlin, 就马上报错: 
![error on kotlin](img/image-20230122120300-2tgbako.png)

原因嘛其实简单, 就是因为kotlin新加了不少语法, 所以在infer类型时会有更多考虑因素, 所以更有可能因信息不够而Infer失败. 所以这里你需要指明更多的信息给kotlin. 所以上面的正确kotlin写法是: 

```kotlin
val numbers = Observable.generate(
    Supplier { this.getCache1() },
    BiConsumer { initValue: Int, emitter: Emitter<Int> -> emitter.onNext(initValue * 2) }
)
```