# 集成测试
如果我们进行一个集成测试, 那我们不用特意为RxJava做什么额外操作. 因为集成测试是测试整个app, 所以rxjava这些属于细节, 并不影响我们的集成测试

# 单元测试
单元测试是直接运行在JVM上的, 而且是"直肠子"(即不考虑异步的). 偏偏RxJava自己就是个异步高手, 所以我们得为这个而做一点额外工作, 才能对有RxJava的ViewModel等类进行测试. 

## step 2A. 使用TestScheduler
TestScheduler就像是Schedulers.io(), AndoridMainScheduler一样, 是一个特殊的, 专为测试而定制的TestScheduler.

类的声明是: 
```kotlin
import io.reactivex.rxjava3.schedulers.TestScheduler
```

它的主要作用是: 它能手动操作时间, 如
```kolint
testScheduler.advanceTimeBy(5L, TimeUnit.MINUTES) //当我要测试一个音乐app的"定时关闭"功能时
```

## step 2B. 使用Schedulers.trampoline()
