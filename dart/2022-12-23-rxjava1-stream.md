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

