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
      child: const Text("work")
    ),

}  

// 4. GetxController for one specific page
class MergeController extends BaseRxCtrl {
  void work(){
    Rx.concat([stream1, stream2]) 
        .listen(print)
        .clearBy(disposables);
  }

}
```

```