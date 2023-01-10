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
