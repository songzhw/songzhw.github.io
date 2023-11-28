# 集成测试
如果我们进行一个集成测试, 那我们不用特意为RxJava做什么额外操作. 因为集成测试是测试整个app, 所以rxjava这些属于细节, 并不影响我们的集成测试

# 单元测试
单元测试是直接运行在JVM上的, 而且是"直肠子"(即不考虑异步的). 偏偏RxJava自己就是个异步高手, 所以我们得为这个而做一点额外工作, 才能对有RxJava的ViewModel等类进行测试. 

## step 1A. 使用TestScheduler
TestScheduler就像是Schedulers.io(), AndoridMainScheduler一样, 是一个特殊的, 专为测试而定制的TestScheduler.

类的声明是: 
```kotlin
import io.reactivex.rxjava3.schedulers.TestScheduler
```

它的主要作用是: 它能手动操作时间, 如
```kolint
testScheduler.advanceTimeBy(5L, TimeUnit.MINUTES) //当我要测试一个音乐app的"定时关闭"功能时
```

## step 1B. 使用Schedulers.trampoline()

在这里就要先介绍一下, 什么是trampoline的scheduler.

Schedulers.trampoline()有两个特点 :
* 它其实跟线程切换无关, 它就是发生在当前线程
* 但它会把工作都放到一个先进先出(FIFO)的队列里, 来一个个地执行.

所以RxJava中的官方介绍就是: Schedulers.trampoline()自己就是 `a Scheduler that queues work on the current thread`


## step 2. 使用自定义的Test Rule
JUnit的Test Rule有点类似 Android中Activity的lifecycle, 可以做一些生命周期相关的工作. 所以我们有这样一个rule, 它接收scheduler为参数.
* 若你想快进时间, 就传入TestScheduler
* 若你想把异步变同步, 那就传入Schedulers.trampoline()


```java
class RxJavaTestRule(private val scheduler: Scheduler = Schedulers.trampoline()) : TestRule {
	override fun apply(base: Statement, description: Description?): Statement {
		return object : Statement() {
			override fun evaluate() {
				RxJavaPlugins.setIoSchedulerHandler { scheduler }
				RxJavaPlugins.setComputationSchedulerHandler { scheduler }
				RxJavaPlugins.setNewThreadSchedulerHandler { scheduler }
				RxAndroidPlugins.setInitMainThreadSchedulerHandler { scheduler }
				RxAndroidPlugins.setMainThreadSchedulerHandler { scheduler }

				try {
					base.evaluate()
				} finally {
					RxJavaPlugins.reset()
					RxAndroidPlugins.reset()
				}
			}
		}
	}
}
```

## step 3. 使用它

```kotlin
class SomeViewModelTest {
  @Rule @JvmField val rule = RxJavaTestRule()
  // 或是使用 @Rule @JvmField val rule = RxJavaTestRule(TestScheduler())
  ... ...
}
```

# 题外话 Schedulers.trampoline()的另一使用场景 
网上有人说过, 他们接入的某SDK的并发不行, 结果同时多人操作时, 结果很混乱 

于是他的做法就是利用Schedulers.trampoline().  这个Schedulers.trampoline()自己内部维护了一个先进先出(FIFO)的队列, 这样整个结果就很正常了

```kotlin
sdk.doSomething()
  .subscribeOn(Schedulers.trampoline())
  .subscribe { .... }
  .clearBy(disposables)
```

# 题外话: 其它Scheduler

下面就是几种不同的scheduler
```kotlin
  Scheduler.io()
  Scheduler.computation()
  Scheduler.trampoline()
  Scheduler.newThread()
  Scheduler.single()
  Scheduler.from(executor)
```
