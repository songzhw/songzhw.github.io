# Introduction of RxDart
This is a new series of RxDart. RxDart used  to have Observable class. But since `RxDart v0.23`, the RxDart has removed the `Obseravble` class, and use `Stream`'s extension instead. More details will be covered afterwards.

As the [RxDart's document](https://pub.dev/packages/rxdart#rx-observables-vs-dart-streams) says, RxDart extends the capabilities of Dart Stream and StreamController. So we have to introduce the Dart's Stream and StreamController first.

# Dart's async code

## Future
If you are familiar with JavaScript, then Dart's `Future` is actually JS's `Promise`. 
Also just like JS, any function marked with `async` will return a `Future` object. 

```dart
  Future<int> getFuture() async{
      final f1 = await Future<int>.delayed(Duration(seconds: 4), () => 43);
      return f1;
  }

  getFuture()
      .then((value) => print('result = $value'))
      .catchError((err) => print('error = $err'))
      .whenComplete(() => print('completed'));
```

## Stream
A `Future<T>` instance will produce a value of type T sometimes in the future. 
If you want to produce a series of data in the future, then what you need is `Stream<T>`  object.
p.s. `Stream<T>` is kind of similar with the JS's `generator`, or Kotlin's `Sequence`. 

Also, any function marked with `async*` will return a `Stream` object.

Here is an example of optional map -- if map function is provided then do the map, if not then do nothing: 
```dartt
// 1. async* functions will return Stream objects
Stream<T> myOptionalMap<T>(Stream<T> source, [T Function(T)? map]) async* {
  if (map == null) {
    yield* source; // 2. yield* will generate another stream
  } else {
    await for (final datum in source) { // 3. await for-in will iterate over a Stream object
      yield map(datum); // 4. yield will put one datum to stream
    }
  }
}
```

Another example: 
```dart
main() async {
  var stream = Stream<int>.periodic(const Duration(seconds: 1), (value) => value).take(3);
  await for (final element in stream) {
    print(element);
  } //=> 0,1,2
}  
```
p.s. To use `await` inside, the main function need to be marked as `async`, otherwise there is no waiting when iterating over the stream.

## StreamController
`StreamController` is actually trying to help us create and control streams. It helps us to generate data without the `yield` by using `add(T)`, also it can close the stream by using `close()` and `isClosed`. 


```dart
final controller = StreamController<int>();
controller.add(23);
controller.add(11);
if (!controller.isClosed) {
  controller.close();
}

var stream = controller.stream;
stream.listen((int value) {
  print('$value'); //=> 23, 11
});
```

## Broadcast
By default, Stream can only have one listener/observer/downstream. If you have more than one listener, then your app will crash due to `Bad state: Stream has already been listened to` error. 

```dart
  final ctrl = StreamController();
  ctrl.add(20);
  ctrl.add(13);

  ctrl.stream.listen((event) => print('Event: $event'),);

  // 这里会crash, 报错: Bad state: Stream has already been listened to.
  ctrl.stream.listen((event) => print('second listener: $event')
```

If you do have the need to multiple listener, then you have to use `asBroadcastStream()` function to convert a default stream to a broadcast stream.
```dart
main() {
  final ctrl = StreamController();
  ctrl.add(20);
  ctrl.add(13);

  final bc = ctrl.stream.asBroadcastStream();
  bc.listen((event) => print('A: $event'),); //=> A20, A13

  // no crash anymore
  bc.listen((event) => print('B: $event'));  //=> B20, B13
} 

```