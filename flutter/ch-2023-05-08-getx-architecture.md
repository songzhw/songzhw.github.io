## 前言

### 架构

这里讲的架构并不是MVC, MVP, MVVM之类的.
其实像React Native, Flutter这种声明式UI, 架构上天生就是MVVM. 当然你还可以搞点花样, 加上[Redux](https://redux.js.org/introduction/getting-started), 弄成个MVI也行, 只不过我三年的React Native经验告诉我, Redux不太好用. 所以我使用MVVM就行了.

这里讲的架构, 主要是充分利用Getx来架构整个App, 更倾向于整体架构上的一些细节, 而不是着重在MVP这样的分层.

### Getx

[Getx](https://pub.dev/packages/get)是一个很强大的库, 好像也是pub.dev上like数最多的一个库.


![image](_image_/image-20230508162836-f7gjl3v.png)

既然有这么多人Like它, 那我就也来分享一下我的使用经验吧.

Getx自己主要是分成四部分的:

* Router (路由, 跳到某页去)
* DI (依赖注入, 类似Dagger, Koin)
* State管理 (管理widget的state, 类似RN中的Redux, 或Android中的Presenter/ViewModel)
* Utilty (各种工具类, 工具方法)
  下面的讲解也是围绕这几部分来的

## 二. 路由

### 1. 为何不用Flutter自己的Router系统

Flutter[自己是有router系统的](https://docs.flutter.dev/ui/navigation#using-named-routes),  但缺点也明显
1). 这个自带的router的named router有局限性, flutter并不推荐用. 但是named router明显能快速对接deep links, 所以这种局限性很不利于我们开发

2). 功能有限. 像我们需要的off, offAll这些功能就没有 (getx里有)

3). 使用时还需要有一个context实例.
但我们并不是随时随地都持有一个context的, 这也局限了我们的使用场景.

4). 使用起来麻烦

```
// 这是Flutter自带的router系统  
onPressed: () => Navigator.of(context).push(  
    MaterialPageRoute(  
        builder: (context) => const SongScreen(song: song)
    ),  
);

// 这是Getx  
onPressed: () => Get.to( SongScreen() );
```

所以我还是推荐使用Getx的Router系统

### 2. 路由 (Router)

因为这毕竟不是一篇介绍Getx的入门文, 所以这里就不多做Getx的讲解, 只做一些关键的说明, 或是架构上的说明.

### 2.1 定义路由

我们一般要先定义一个Getx的Router. 因为我们要对接deep link, 即后台给我们一个`yourcompany://page1?id=23`的string, 你能跳到某一页去 (并带上参数id=23). 所以我们的基本要求就是: **要能通过一个plain string就能知道要跳到哪个页面去, 并支持传参**

```dart
GetMaterialApp(  
  initialRoute: "/home",  
  unknownRoute: GetPage(name: "/404", page: () => const NotFoundPage()), 
  routingCallback: (routing) { ... } //相当于跳转的监听器
  getPages: [  
    GetPage(name: "/home", page: ()=> const HomePage()),  
    GetPage(name: "/detail", page: ()=> OneDetailPage()),
```

* `initialRoute`是首页是哪个
* `unknownRoute`是没找到对应的页面时, 就去这个页面
* `getPages`定义一个Map<String, function>的路由表. 注意page都是函数, 这样我们就能做到lazy initialization.
  要是用 `GetPage(name: "/home", page: HomePage())`这样的路由表, 那一打开app, 所有页面都初始化好了, 太浪费资源, 也太慢了.
* `routingCallback`是跳转的监听器,  当你跳转到下一页, 或是按back等会调用它. 详情可见[这里](https://github.com/jonataslaw/getx/blob/master/documentation/en_US/route_management.md#middleware).

#### 坑1

很多从web转过来的人, 都喜欢让initRouter定义为"/". 但在Getx中, 这样做会让unkonwRoute失效. 这是我经过反复研究才找到的一个隐藏bug吧.

也就是说, **为了unknowRoute能成功, 你的initRoute不能是&quot;/&quot;.**

### 2.2 跳转

这里的跳转功能就很丰富了, 下面一一讲解

```dart
Get.to(NextScreen());  
Get.toName("/detail"); //比起to(), 一般使用toNamed()

// 跳转时带参数 
Get.toNamed("/router/back/p3?id=100&name=flutter")

// 本页finish, 再跳detail页
Get.offNamed("/detail"); //其实就是说用detail页来replace掉本页

// 使用场景: 当点了"logout"按钮时, 清除所有页, 再跳到login页去
Get.offAllNamed("login"); //类似Android中的clear_task | new_task

// 后退
Get.back();

```

注意到Getx的路由跳转是不需要context的, 这样你在任何地方的代码(如ViewModel, Repository, ...)都可以写跳转的代码.

#### 跳转时的传参

当然, 若你的参数是非String, 是个普通类, 那就得用`argument`, 而不是`parameter`

```dart
// A1). 带Map<String, String>参数  
final params = <String, String> {"source": "p1", "value": "230"};  
Get.toNamed("/router/back/p2", parameters: params);  
// 或用这种方式也行  
Get.toNamed("/router/back/p3?id=100&name=flutter")

// A2). 下一页中取出Map<String, String>参数  
String source = Get.parameters["source"] ?? "<default>";  
String name = Get.parameters["name"] ?? "<default>";

// - - - - - - - - - - - - - - - - - - - - - - - -

// 若参数不是Map<String, String>类型, 就用不了parameter  
// 这时就要用 Get.toNamed("..", argument)

// B1). 存入值  
final params = Offset(12, 13);  
Get.toNamed("/router/back/p3", arguments: params);

// B2). 取出值  
final args = Get.arguments;
```

## 三. 依赖注入 (DI)

Getx的DI主要就是 `Get.put(obj)`, 然后取出来用`obj = Get.find()`. 这样就能存对象, 取对象了.

初看好像很简单, 但其实是有坑的, 特别是在使用Binding时.

### 3.1 Binding

现在假设我们有两个页面, 一个是Home页展示各种商品, 另一个Detail页展示一种商品的详情.
在Getx中我们可以使用Binding, 这个Binding类似Dagger中的module, 或是Koin中的module, 即提供对象的.

```dart
class HomeBinding implements Bindings {
  @override
  void dependencies() {
    Get.lazyPut<HomeController>(() => HomeController());
    Get.put<Service>(()=> Api());
  }
}

class DetailsBinding implements Bindings {
  @override
  void dependencies() {
    Get.lazyPut<DetailsController>(() => DetailsController());
  }
}
```

这样你可以在路由系统中加入binding, 这样跳入到home页与detail页时就能自带上面的HomeController, Service, DetailsController这些对象了.

```dart
Get.to(Home(), binding: HomeBinding());

//或是:
Get.to(DetailsView(), binding: DetailsBinding())
```

### 3.2 Binding的缺点

上面的写法, 其实有两个很大的缺点.

##### 第一个缺点

`Get.to(widgetObj, bindings)`是可以注入binding.

但是`Get.toNamed()`并不支持binding参数啊. 我的跳转一般都是用toNamed的, 所以注定了这种方式我用不了.

##### 第二个缺点

这个缺点很隐藏, 很容易出问题. 以上面的binding为例

* `HomeBinding`中提供了 HomeController, Service 两个对象
* `DetailsBinding`中提供了 DetailsController 对象
  但其实我们的Details页中也会用到Service对象.

之所以不出现"details页中说找不到Service"的crash, 是因为用户先打开的home页, Home已经往Get中写入了Service对象了, 所以等之后打开detail页时, serivce对象已经有了, 能够`Get.find()`得到, 所以不会有NPE错误.

但要是deep link的场景呢?

: 你直接跳到了Detail页, 结果就因为没有经过home页, 所以`Service service = Get.find()`找不到service对象, 应用会crash.

所以现在就明白了, 第二个缺点就是: 上面两个Binding有隐藏的依赖性 DetailsBinding其实依赖于HomeBinding. HomeBinding不先放好service, 那DetailsBinding提供不了Serivce, 就可能会让Detail页crash.

### 3.3 全局Binding

也就是像Dagger或Koin一样, 一开始就定义好一个全局的"对象提供表", 即Dagger与Koin中讲的`Module`啦.

这样好处是:
1). 因为是全局的, 所以我们使用`Get.toNamed("...")`也能使用到全局提供的对象

2). 因为是全局的, 所以没有什么Binding1依赖于Binding2的问题. 总共就一个Binding嘛, 自然没什么依赖的前后关系问题.

#### 定义全局Binding

```dart
GetMaterialApp(
  initialBinding: AppDiModule(), // 就是定义这个
  ... 
);

class AppDiModule implements Bindings {
  @override
  void dependencies() {
    Get.lazyPut(() => AppleRepository(), fenix: true);
    Get.lazyPut(() => BoxService(), fenix: true);
    Get.lazyPut(() {
      BoxService service = Get.find();
      return BoxRepository(service);
    }, fenix: true);
  }
}
```

#### 具体业务页面中取出值

这个就容易了, 直接使用`Get.find()`就行了. 如:

```dart
class DiPage2 extends StatelessWidget {
  @override Widget build(BuildContext context) {
    HoeService service = Get.find();

class DiPage3 extends StatelessWidget {
  @override Widget build(BuildContext context) {
    HoeRepository repo = Get.find();
    final service = repo.service;
```

## 四. state管理

好了, 这个是我喜欢Getx的一点.  不过在先说之前, 先得说我最讨厌React Native的一点

### 声明式UI框架的一个普遍问题

像Flutter, React Native这些声明式UI框架, 我碰到过的性能问题, 主要是两点

1). 这些声明式UI全是UI框架, 一涉及到非UI的东西, 就要走Android, iOS端. 这时的来回传数据, 可能会有性能问题.

-- 当然, 这个不绝对. 因为我在做一个React Native项目时, 我用crypto-js去解密一个作品时, 很慢. 但当我下沉到Android, iOS端去解密, 反而性能大大提升了. 所以还是要看使用场景.

2). `setState()`式的刷新
也就是说你一个TextView要刷新了, 结果我们调用setState却是刷新整个页面. 这一点在React Native里更加明显, 特别是超长的list列表时, 性能超级差.

题外话: 因为这个setState的原因, 相对于React, 我其实更喜欢[Solid JS](https://www.solidjs.com/), 这个SolidJS会知道你要刷新哪个组件, 而去精确刷新某一组件, 而不是刷新整个页面, 但效率自然更快了

![image](_image_/image-20230508171249-4524wif.png)

### 4.2 Getx对性能的提升

Getx就像Solid JS一样, 能精确刷新某一个需要刷新的组件, 而不是刷新整个页面, 所以你的Flutter app效率自然就更好了.

备注: 因为不需要刷新整个页面, 所以在使用Getx时, 完全可以不用StatefulWidget. 仅仅使用StatelessWidget基本上就够了.

#### 基本使用方式

```dart
// 两种声明方式  
final name = "".obs;   //声明方式1, 使用obs
Rxn<ui.Image> imgSrc = Rxn<ui.Image>(); //声明方式2, 使用Rx或Rxn. 

// 使用:  
Obx( ()=> MyWidget(imgSrc.value) )

// 更新  
name.trigger(newValue)
```

备注: `Rxn`可以不提供初始值, 这在一些异步场景中就比`Rx`更好用了.

### 4.3 Getx的state管理的先进性

讲过最大的好处之后, 我们现在来全面地看一下Getx的statet管理的各种好处.

现在的flutter一些state管理库, 要么像Bloc一样使用很复杂, 要么像Mobx一样要用代码生成(这很慢), 所以Getx想要快一点, 使用更简单的state管理.

* 说它简单, 是因为你不要使用什么StreamController, StreamBuilder, 或为一个状态专门创建一个类. 直接用`"name".obs`中的obs就能创建一个可监听的值

  * 而且你也不用像React一样, 自己定义`memo( oldProps, newProps => ...)` 来自己定义要如何比较. Getx会自己比较`Obx(()=>widget)`中的值的前后变化, 来决定是否要更新的.
* 说它快, 是因为它不用代码生成. flutter中的codegen真的很慢
* 另外, 最大的一个特点就是, 用了Getx的state管理之后, 你再也用不着StatefulWidget了. 仅仅StatelessWidget就够你用了! 性能自然也提升很多!

### 4.4 GetxController

这个就有点类似ViewModel, Presenter或Controller类. 你的那些数据以及操作数据的方法都在这里, 如:

```dart
class MyResearchCtrl extends GetxController {
  Rx<Color> color = Rx(Colors.indigo);
  void setColor(Color c) => color.trigger(c);
}
```

这样你就能在多个页面中使用, 甚至是共享这个GetxController:

```dart
// 共享
class ControllerAndPage3 extends StatelessWidget {
  @override Widget build(BuildContext context) {
    final ctrl = Get.put(MyResearchCtrl());  


class ControllerAndPage2 extends StatelessWidget {
  @override Widget build(BuildContext context) {
    final ctrl = Get.find<MyResearchCtrl>();  
```

### 4.5 生命周期

上面讲过Getx多是使用StatelessWidget. 但麻烦就来了, 即StatelessWidget没有生命周期方法

而StatefulWidget是有生命周期方法的, 其实是它的State有啦.  它的`initState`与`dispose`方法就是生命周期的开始与结束), 那碰到这种要在页面打开时注册某一东西, 然后在页面退出时注销掉什么(以免内存泄露)时, 要怎么办?

: 不用担心, GetxController就有生命周期, 这样就能代替StatefulWidget的生命周期.

‍`dart class MyCtrl extends GetxController{   @override void onInit() {...}   @override void onClose() { ...} } ‍`

## 结语

Getx提供了

* 更强大的Router系统
* 更加提升性能的State管理
* 更多Utils方法 (如求屏幕宽高, 如不用context就能显示dialog, ...)
  所以真的推荐使用Getx来架构你的app.

当然Getx还有DI系统, 只不过感觉还行, 只不过不是这么惊艳.
备注: 依照我个人喜欢, 其实我更喜欢用[flutter-koin](https://pub.dev/packages/koin). 因为它简单, 好用, 还和我以前在Android中使用的koin一脉相承. 不过Getx的DI也还行, 只不过没有Koin中的`factory`, `single`这样好用而已.