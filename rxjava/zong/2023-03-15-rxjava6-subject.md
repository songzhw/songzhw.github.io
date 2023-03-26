这一章一开始先要涉及到RxJava中的理论部分(单播与多播). 但纯讲理论太虚, 所以我会穿插一些实例, 来让讲解贴近生活, 就不会显得太枯燥.  接下来就会讲到多播以及多播中的Subject. 

# 单播(unicast)
若你以前听说过`冷Observable`与`热Observable`, 那很好. 这里我就先说好, 其实`冷Observable`就是单播的. 所以你可以理解这一大节就是要讲`冷Observable`. 
若你没有听说过冷热Observable,没事, 后面我也会讲到这两个概念. 


## 单播的概念
说回unicast, RxJava中有两种播放形式
* 多播(multicast) : 一个上游可以有多个下游时, 这就是多播. 即面向多个下游播放数据. 
  * 典型代表就是 `Subject`的子类们
* 单播(unicast) : 一个上游就只有一个下游, 这就是单播. 
  * 所有除了`Subject`的流, 基本上都是单播 (都是cold Observable) 

上面讲了, 除了`Subject`的流, 基本上都是冷流. 那`Observable.interval(1s)`也是冷流, 也是单播, 是吗? 
: 是的, 没错, 是冷流, 是单播

这样说来, 那就不对了, 我明明可以对`* Observable.interval(1s)`进行多个下游注册, 如这样: 
```kotlin
    val src_ = Observable.interval(500, TimeUnit.MILLISECONDS).take(5)
    src_.subscribe { count -> println("szw(A) : $count") } //=> 0,1,2,3,4
    src_.filter { it % 2 == 1L }.subscribe { count -> println("szw(B) : $count") } //=> 1, 3
```
上面的代码, 即不会编译出错, 也能运行正常并得到结果. 这样一个`src_`, 明明是单播, 怎么它就能有两个下游呢? 
: 好问题. 这其实就是大多数人对RxJava中"单播与多播"最大的疑惑了. 
我先说结论哦, 你可以这样理解, `每当你subscribe一次, 冷Observable都会copy一次上游. 这样两次subscribe, 本质上仍是两个上游. 本质上仍是一对一的单播`. 

## 单播的例子
上面的代码可能不好验证. 我们来换一个更好验证上面结论的做法. 即用RxJava发出网络请求, 然后能过抓包的方式就清楚了. 

比如说: 
```kotlin
interface UserService {
    @GET("4d5ade01-ded4-44c5-b6db-8a2ee9ea05c2")
    fun getUsers(): Observable<E1>
}
```

然后我们来注册两个下游, 并调用这个请求: 

```kotlin
source = retrofit.create(UserService::class.java)
    .getUsers()
    .schedules()

btnRequestUser.setOnClickListener {
    source
        .subscribe { resp -> /*刷新列表*/ }
        .clearBy(disposables)
    source
        .subscribe { resp -> /*更新"共有N条数据"的TextView*/ }
        .clearBy(disposables)        
}
```

这样当我们点击按钮时, 我们通过Android Studio的Network Inspector来看一下, 结果惊奇地发现, 我明明只调用`getUsers`一次, 但却发了两次网络请求: 
![image](img/image-20230322094831-7etdppt.png)

这个例子就完美地展示了, 为何明明可以有subscribe多次, 却仍是单播了. 
: 因为你每次subscribe, 都是对应了一个新上游. 严格来说, 每个上游仍是一一对应了一个下游而已. 所以这就是单播. 

当然, 你可能还会好奇, 要怎么样才能解决上面这样冗余的网络请求呢? 
答案其实很简单, 就是把单播变为多播, 这样一个上游能有多个下游, 就只会发出一次网络请求了. 至于什么是'多播', 如何把单播变为多播, 下面的章节就是介绍了. 

# 多播
多播就是一个上游可以有多个下游. 同样, 我们经常提及到的"hot Observable", 也是指多播. 至于冷与热的Observable, 我们后面再讲. 

而讲解多播, 因为网上资料比较少, 我不得不借鉴RxJS中的多播概念来讲解. 其实RxJS, RxJava中的多播概念是一模一样的, 但RxJS的因为有multicast这个操作符(RxJava中没有)