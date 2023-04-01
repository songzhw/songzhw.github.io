

# Flutter里built-in的native开发
Flutter自己, 和React Native一样, 仅仅是一个UI框架而已. 这跟Android, iOS系统还是差了很多. 也就是说, 当涉及到: 指纹, 地理位置, 设备文件, 拍照, ... 等等诸多非UI的工作时, Flutter自己是处理不了的. 

但是Flutter提供了一个跨平台框架, 叫做`MethodChannel`, 来下沉这种非UI的工作到native平台去. 说人话就是, Flutter自己不支持拍照功能, 但Android, iOS支持啊. 于是当用户在Flutter中想拍照时, Flutter就告诉Android或iOS, 说"请你拍照, 拍完了告诉我".  这样当native平台干完了活, 就告诉Flutter完成结果(成功的数据, 或失败的原因). 这种就是所谓的"下沉到native平台"的工作. 

原生的MethodChannel还蛮麻烦的, 我举个例子哦, 来看下为了支持某一功能,  Android端要这样接收来自Flutter端的请求: 
```kotlin
    val methodChannel = MethodChannel(flutterEngine.dartExecutor.binaryMessenger, "CHANNEL_NAME")
    methodChannel.setMethodCallHandler {
      // This method is invoked on the main thread.
      call, result ->
      if (call.method == "getBatteryLevel") {
        val batteryLevel = getBatteryLevel()

        if (batteryLevel != -1) {
          result.success(batteryLevel)
        } else {
          result.error("UNAVAILABLE", "Battery level not available.", null)
        }
      } else {
        result.notImplemented()
      }
    }
```

有经验的开发, 一眼就能看了这个代码最大的问题: `扩展性太差`!
要是以后有一堆的native功能需要通过MethodChannel来提供, 那代码就会慢慢地变得臃肿, 比如像这样: 
```kotlin
methodChannel.setMethodCallHandler { call, result -> 
    if(call.emthod == "getBatteryLevel") {...}
    else if(call.emthod == "getLocation") {...}
    else if(call.emthod == "takePicture") {...}
    else if(call.emthod == "fingerprintVerify") {...}
    else if(call.emthod == "getScreenLighting") {...}
    else if(call.emthod == "getOSLanguage") {...}
    else if(call.emthod == "getAppCachePath") {...}
    ... ...
    ...
}
```
不用说这样一长串的if-else chain是很不利于维护, 也容易出错的. 在代码分工上, 我们需要代码解耦合, 各自只负责自己的部分, 而不是让上面一个类越来越大, 什么都管.

另一个MethodChannel的问题就是, 不区分数据类型. 就是说, 数据类型在Flutter, Android, iOS中是不一样的. 但因为跨平台了, 所以MethodChannel并没有提供一个机制来检验你传来的到底是不是String, 到底是不是MyResponse类型. 这样当类型不对时就会crash.  更好的体验则是, 像java等强验证语言一样, 在dev编码时就告诉你这个类型不对, 而不是等到运行时了让用户crash. 


# Pigeon的工作原理(简略版)
所以Pigeon就是为了解决这两个问题而产生的. Pigeon其实就是让你自己定义好了你这次工作的接口, 然后Pigeon帮你生成MethodChannel的代码. 这样整体代码看起来简洁, 分工好, 而且是type safe. 

比如说当你定义了一个接口: 
```dart
import 'package:pigeon/pigeon.dart';

class Book {
  String? title;
  String? author;
}

@HostApi()
abstract class BookApi {
  List<Book?> search(String keyword);
}
```
-- 这样每个不同的功能, 就是独立一个Api类, 这样是不是比长长的if-else chain要解耦得多

而你运行一个脚本(下面会讲), 就会为我们在Flutter端, 在Android端, 在iOS端生成代码. 以Android端的为例, 它就是这样的基本结构: 

```kotlin

data class Book (
  val title: String? = null,
  val author: String? = null
)

interface BookApi {
  fun search(keyword: String): List<Book?>

  companion object {
    fun setUp(binaryMessenger: BinaryMessenger, api: BookApi?) {
        val channel = BasicMessageChannel<Any?>(binaryMessenger, "dev.flutter.pigeon.BookApi.search", codec)
        if (api != null) {
          channel.setMessageHandler { message, reply ->
            ...
            reply.reply(wrapped)
          }
        } 
    }
}

```

看到了这个生成的代码, 就知道Pigeon其实也不是什么黑科技, 就是底层使用了MethodChannel而已.  `(备注: MethodChannel与BasicMessageChannel是类似的工具, 只不过BasicMessageChannel还有相关的解码编码集而已. 你可以理解二者是近似的)`

备注: 其实Android程序员应该很熟悉. MethodChannel就是类似OkHttp, Pigeon就是类似Retrofit (也是定义好接口, 为开发生成代码, 最终底层都是MethodChannel/OkHttp在工作)


# Pigeon开发前的准备工作
下面就开始是本篇文章的重点了. 主要Flutter与pub.dev上的文档是相互冲突, 或是根本就不齐全的. 
* Flutter官网的 [Writing custom platform-specific code](https://docs.flutter.dev/development/platform-integration/platform-channels?tab=type-mappings-obj-c-tab)
* Pigeon自己的[ReadMe与Exampel](https://pub.dev/packages/pigeon/example).

特别是要去运行iOS端, 就会碰到一些xcode相关问题, 这个真的是坑, 所以要一一讲解下这些坑以及如何绕过这些坑. 

## step 1. 安装Pigeon

```js
$ dart pub add pigeon  
// 2023.03年安装了 pigeon v9.1.3(现已经支持swift与kotlin了)
```

以前我用pigeon v8.x时, 还只支持生成java与objc代码.  现在v9.x时代已经支持swift与kotlin了.

## step 2. Flutter端 - 定义接口文件
```dart
// BookIn.dart
import 'package:pigeon/pigeon.dart';

class Book {
  String? title;
  String? author;
}

@HostApi()
abstract class BookApi {
  List<Book?> search(String keyword);
}
```

## step3. Android端
若你的Android端的主包是`a.b.c`, 而你想Pigeon生成后的代码放到`a.b.c.pigeons`包里, 那很容易, 你去Android Studio等IDE新建目录(directory)或是新建包(package)都行, 总之就是要先建立好`pigeons`这个包. 不然下面的脚本会报错, 说目标目录不存在, 无法新建文件

## step4. iOS端 (坑1)
**这一步是Flutter与pub.dev官网上都没讲到的一步. 不加这一步, 就编译iOS端会失败. 所以这算是官网没讲清楚, 导致失败的第一个坑. **

你的iOS端默认就是把swift文件放到`Runners`目录下的, 如下面的AppDelegate.swift文件一样. 

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/17f82348983c47b9bfd61c9d9ccdda2f~tplv-k3u1fbpfcp-watermark.image?)


自然, 为了管理起来更方便, 我们不可能什么都放到Runners目录下. 类似Android中的package, iOS端的包叫做`group`. 

1). 我们在xcode中打开Flutter工程的`iOS`目录中的`Runners.xcworkspace`:

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bb35d9d1b10040448e71915db301f5a0~tplv-k3u1fbpfcp-watermark.image?)


2). 在Runners上新建group, 取名叫`pigeons`:
![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/467ae578f6ce4b98b6c906aec1e07d9d~tplv-k3u1fbpfcp-watermark.image?)

3). 生成group后, 在`pigeons`这个gorup上右击, 选择新建swift文件, 新建一个`BookGenerated.swift`文件

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/00724d39ed674baf8226389482ca5dbe~tplv-k3u1fbpfcp-watermark.image?)

总结: 也就是说, 在运行脚本前, 我们必须得
* 在Android端建立好目标目录(即package)
* 在iOS端建立好目标目录(即group), 以及**目标文件**!!!


至于原因, 那就是和Android程序大大的不同, xcode工程有一个索引文件. 
* Android工程中, 你新加一个txt, gradle会知道在build时不加入这个txt文件到build过程中来. 若你在src目录中新加了个java文件, gradle会自动加入这些java文件到build过程中来
* iOS工程则不是. Xcode完全没这么智能, 它根本不会自动地把所有swift与objc文件(.h, .m)加入到编译过程中来. Xcode会去找一个叫`project.pbxporj`的文件. 只有在这个`project.pbxporj`里的文件与目录, 才会加入到编译过程中来. 

也就是说, 官网上说的脚本自动生成swift文件后, xcode根本不认识这个新文件, 从而会导致最终编译失败, 说找不到`BookApi`这个类型. 
解决办法就是上面的: 你自己去xcode中找开iOS工程, 新建group与swift文件. 这样一来, `project.pbxporj`就会把刚刚新建的group与swift文件加入进来. 
p.s. 后面你运行脚本后, swift文件的内容会被脚本的生成所覆盖, 这个不用担心. 

同时注意,  在git提交中也提交这个文件哦: 

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bd89decb5e9743ec973130590936cf37~tplv-k3u1fbpfcp-watermark.image?)



### 题外话: xcode小知识
`xcodeproj`文件与`xcworkspace`文件都可以打开一个iOS工程. 但是一般使用`xcworkspace`. 原因是: 
* 若xcode新建一个iOS工程, 它会自动生成一个`xcodeproj`文件, 你双击它就可以打开工程
* 但你要是用了cocoapod(一个类似Android端的Gradle的依赖管理工具), pod就会自动帮我们生成`xcworkspace`. 这时为了能使用你在cocoapod中声明的第三方库, 就得用`xcworkspace`来打开iOS工程才行




## step 5. 运行脚本
```
# flutter pub run pigeon \ 
--input lib/pigeons/demo/BookIn.dart \ 
--dart_out lib/pigeons/demo/BookOut.dart \ 
--swift_out ios/Runner/pigeons/BookGenerated.swift \
--kotlin_out ./android/app/src/main/kotlin/ca/six/readerf/reader_flutter/pigeons/BookGenerated.kt \  
--experimental_kotlin_package "ca.six.readerf.reader_flutter.pigeons"​
```

我的BookIn.dart已经在step2中定义好了. 位置就在: `lib/pigeons/demo/BookIn.dart`

而生成的文件, 分别在dart, android, iOS端: 
* lib/pigeons/demo/BookOut.dart
* ios/Runner/pigeons/BookGenerated.swift
* android/app/src/main/kotlin/ca/six/readerf/reader_flutter/pigeons/BookGenerated.kt


最后结果如下: 
![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3063d02c961e464ebc78789756140f33~tplv-k3u1fbpfcp-watermark.image?)


## step 6. Android端连接好Pigeon
```kotlin
import io.flutter.embedding.android.FlutterActivity
// 下面三行import要新加
import io.flutter.embedding.engine.FlutterEngine 
import ca.six.readerf.reader_flutter.pigeons.Book
import ca.six.readerf.reader_flutter.pigeons.BookApi

class MainActivity : FlutterActivity() {
    // 不再重写onCreate(), 改为重写configureFlutterEngine()方法
    override fun configureFlutterEngine(flutterEngine: FlutterEngine) {
        super.configureFlutterEngine(flutterEngine)
        val messenger = flutterEngine.dartExecutor.binaryMessenger
        BookApi.setUp(messenger, BookApiImpl()) // 注意是setUp()方法, 不是setup()
    }
}


class BookApiImpl : BookApi {
    override fun search(keyword: String): List<Book?> {
        val book = Book("android-$keyword", "szw2")
        return listOf(book)
    }
}

```

## step 7. iOS端连接好Pigeon
```swift

// 新加了此类
class BookApiImpl: NSObject, BookApi {
  func search(keyword: String) -> [Book?] {
    let result = Book(title: "\(keyword)'s Life", author: keyword)
    return [result]
  }
}


@UIApplicationMain
@objc class AppDelegate: FlutterAppDelegate {
  override func application(
    _ application: UIApplication,
    didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
  ) -> Bool {
    GeneratedPluginRegistrant.register(with: self)
      
    // 新加了这两行
    let controller : FlutterViewController = window?.rootViewController as! FlutterViewController
    let api = BookApiImpl()
    BookApiSetup.setUp(binaryMessenger: controller.binaryMessenger, api: api)
      
    return super.application(application, didFinishLaunchingWithOptions: launchOptions)
  }
}

```



## step 8. Dart端调用native端干活

```dart
import 'package:reader_flutter/pigeons/demo/BookOut.dart';

  Future<void> nativeSearchBook() async {
    BookApi api = BookApi();
    List<Book?> books = await api.search("from12");
    print('szw reply: ${books[0]}');
  }
```

我们在Flutter里只要调用下这个`nativeSearchBook()`方法就能得到book列表了. 

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ce1b2697d29044de8b0465d3da84f804~tplv-k3u1fbpfcp-watermark.image?)


完美打通了三端. 上面的其实都是我摸索中的正常代码, 下面我就来讲下官网上的各种错误, 免得其它人也碰到类似的问题

# Pigeon的坑 -- iOS端

## 坑1
xcode中不会自动把你生成的`BookGenerated.swift`放到`project.pbxproj`文件里去, 所以在运行脚本前我们要先**去xcode中**新建好group与`BookGenerated.swift`

## 坑2
```
Swift Compiler Error (Xcode): Type 'BookApiImpl' does not conform to protocol 'BookApi'
/Users/../ios/Runner/AppDelegate.swift:4:6
```

解决办法就是严格按照生成的protocol(类似Android中的interface)来, 所以只要把下面的左侧代码, 改为右侧代码就行: 

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dc85bdca14ba4108ad3925916839a060~tplv-k3u1fbpfcp-watermark.image?)
(坑就坑在, 左侧是官网上的代码, 你一copy就会编译失败)

## 坑3

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f7f0a2a1164044fb83caf5920eb24de1~tplv-k3u1fbpfcp-watermark.image?)

即pigeon的ReadMe上讲的: 
```swift
let api = BookApiImpl()
BookApiSetup.setUp(getFlutterEngine().binaryMessenger, api)
```
是已经过时了的操作, 要想获得binaryMessenger, 就得用: 

```swift
let controller : FlutterViewController = window?.rootViewController as! FlutterViewController
let api = BookApiImpl()
BookApiSetup.setUp(binaryMessenger: controller.binaryMessenger, api: api)
```


# Pigeon的坑 -- Android端

## 坑4
注意要加import哦
```kotlin
import android.os.Bundle
import ca.six.readerf.reader_flutter.pigeons.Book
import ca.six.readerf.reader_flutter.pigeons.BookApi
```
这个也是官网上没有讲的, 也要自己小心

## 坑5
Pigeon官网上说的是`BookApi.setup()`
但实际上生成的代码却是:  `BookApi.setUp()`, 这个点也要小心

## 坑6
仍是`getBinaryMessenger()`方法已经被删除了. 所以官网上的代码会编译失败: 
```
class MainActivity: FlutterActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // ✯ 坑5: 官网上写是setup(), 但其实应该是 setUp(), 这个真是低级错误, 文档写得太差了
        // ✯ getBinaryMessage()没了, 这是坑6        
        BookApi.setUp(getBinaryMessenger(), BookApiImpl()) 
    }
}
```

真正的写法在上面已经讲过了, 即是: 
```kotlin

class MainActivity : FlutterActivity() {
    // 不再重写onCreate(), 改为重写configureFlutterEngine()方法
    override fun configureFlutterEngine(flutterEngine: FlutterEngine) {
        super.configureFlutterEngine(flutterEngine)
        val messenger = flutterEngine.dartExecutor.binaryMessenger
        BookApi.setUp(messenger, BookApiImpl()) // 注意是setUp()方法, 不是setup()
    }
}
```

# 总结
两个官网的介绍相互冲突, 导致我们开发用起Pigeon来很痛苦. 
另外一个点就同xcode工程要用`project.pbxproj`来管理源文件, 这一点也让很多从Android过来的Flutter开发因为不了解这特性而导致xcode编译失败. 
上面6个坑都填好后, 我们就能成功地运行Android与iOS了