If you happend to, just like me, use `Getx` and `RxDart` in a same project, then you would find out they are not quite compatible with each other. Here is a article to help you tame these two wonderful frameworks. 

## intruduction to these two framework

* `Getx` is a high-performance state management library. It help us not to re-render the whole page when only one subview changes. 
* `RxDart` is a child of the famous Rx framework. Its sibling includes: RxDart, RxSwift, RxKotlin, RxJS, ...

Under the hood, both framework are using observer pattern to listen to some data chagne. That's why we are not surprise to see both framework has a class called `Rx`. 

## Issue 1. the `Rx` Class

```dart
import 'package:get/get.dart';
import 'package:rxdart/rxdart.dart';

class SomePage extends StatelessWidget {
  ...
   Rx.concat([stream1, stream2]) // error here! 
    .listen(print); 
}  
```
Because the `Rx` could be from Getx, also it could be from RxDart. So flutter get confused which Rx is used here. 

In this case, we have to have an alias name to one of the framework. Lucky for us, Dart support alias to the import. Here is the solution: 

```dart
import 'package:get/get.dart';
import 'package:rxdart/rxdart.dart' as RxDart;

class SomePage extends StatelessWidget {
  ...
   RxDart.Rx.concat([stream1, stream2]) // error here! 
    .listen(print); 
}  
```

## Issue 1B. Another approach
If you don't want to write `RxDart.Rx`, you can use the strict import, such as : 
```dart
import 'package:get/get.dart' show Get, Inst, Obx, StringExtension;
import 'package:rxdart/rxdart.dart' show Rx;
```

Then it's okay to use `Obx(() => ... )` and `Rx.combineLatest2(...)` at the same file. 

## Issue 2. how to dispose subscription?
Streams like `Stream.periodic()` would send infinite data, and wouldn't stop it even we exit the page that start this periodic work. If you are familiar with RxJava or RxJS, then what we need is to add a CompositeSubscription to handle the RxDart's subscription, and release it when the page is destroyed. 

## Issue 2.2 no `onDestroy` method in a StatelessWidget
Unfortunately, when we use Getx, we prefer StatelessWidget.
And StatelessWidget has no `dispose` method or `onDestroy` method for us to dispose the subscriptions. 

The solution for it is to use Getx's GetxController, which has a `onInit` and `onClose` lifecycle callback. 

```dart
// 1. Base GetxController
class BaseRxCtrl extends Getx.GetxController {
  final disposables = CompositeSubscription();

  @override
  void onClose() {
    disposables.dispose();
    super.onClose();
  }

}

// 2. Extension method : to clear subscripton more easier
extension ClearRx on StreamSubscription {
  void clearBy(CompositeSubscription mgr) {
    mgr.add(this);
  }
}


// 3. Stateless Widget
class SomePage extends StatelessWidget {
  final ctrl = Getx.Get.put(MergeController());
  ...

  OutlinedButton(
    onPressed: () { ctrl.work(); },
    child: Obx( ()=> Text(ctrl.resp$.value)),
  ),

}  

// 4. GetxController for one specific page
class MergeController extends BaseRxCtrl {
  final resp$ = "".obs; // This is like the LiveData in the
  void work(){
    Rx.concat([stream1, stream2]) 
        .listen((datum) => resp$.value = datum)
        .clearBy(disposables);
  }

}
```

### complete code
```dart
import 'package:flutter/material.dart';
import 'package:get/get.dart' show Get, Inst, Obx, StringExtension;
import 'package:rxdart/rxdart.dart' show Rx;
import 'package:rxdart_demo/BaseRxCtrl.dart';
import 'package:rxdart_demo/Constants.dart';
import 'package:rxdart_demo/ext/RxDartExt.dart';

import '../CommonWidgets.dart';

class ApisDemo extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final ctrl = Get.put(ApisCtrl());
    ctrl.fetchData();

    return Scaffold(
      appBar: appbar("ApisDemo"),
      body: Center(child: Obx(() => Text(ctrl.resp$.value, style: TextStyle(fontSize: 30)))),
    );
  }
}

class ApisCtrl extends BaseRxCtrl {
  final resp$ = "xxx".obs;

  Future<String> f1() async {
    await Future<void>.delayed(oneSec);
    return 'Value1';
  }

  Future<String> f2() async {
    await Future<void>.delayed(twoSec);
    return 'Value2';
  }

  void fetchData() => Rx.combineLatest2(
        Rx.fromCallable(f1),
        Rx.fromCallable(f2),
        (v1, v2) => resp$.value = "v1 = $v1, v2 = $v2",
      )
      .listen((event) => event)
      .clearBy(disposables);
}

```