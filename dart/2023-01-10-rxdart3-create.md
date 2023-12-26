# No more `Observable`
If you are familiar with RxJava or RxJS, you might be used to the use of `Observable` classes. However, don't be surprised if you can't find `Observable` in the RxDart package. That's because since RxDart 0.23.x, it remove all OBservable classes, and use the `Stream`'s extension instead. But no worries, this article, or even this series of articles, will introduce all the valuable details. 

# Create Observable
Rx world has two famous concept:
* `Observable`: create a stream to send out data or error(, also know as `upstream`)
* `Observer`: listen to a stream to receive data or error(, also know as `downstream`, `listener` or `subscriber`)


As we mentioned before, `Observable` is deprecated in RxDart, and is replaced with `Stream`. <br/>
However, sometimes, you would see `Rx` class to provide some operator for us as well. 


This article is mostly about how to create stream, or how to create Observable for us. 

# cheat sheet of creating observables
| RxJS                 | RxJava             | RxDart                                           |
|----------------------|--------------------|--------------------------------------------------|
|    Observable        | Observable, Flowable | Stream, RX                                     |
| create()             | create()           |                    x                             |
| generate()           | generate()         |                    x                             |
| of(1,2,3)            | just(1,2,3)        |                    x                             |
| from([1,2,3])        | fromIterable(listOf(1,2,3)) | Stream.fromIterable([1,2,3])            |
| range(start, count)  | same as RxJS       | Rx.range(start, end)                             |
| repeat(count)        | same as RxJS       | Rx.repeat( index=> Stream.value(index*2), count) |
| repeatWhen(ob2)      | same as RxJS       |                    x                             |
| -----                | -----              | -----                                            |
| timer(time)          | same as RxJS       | Rx.timer(value, duration)                        |
| interval(time)       | same as RxJS       | Stream.periodic(duration, index => xxxxx)        |
| -----                | -----              | -----                                            |
| fromPromise(promise) | fromFuture(f)      | fromFuture(f)                                    |
| x                    | fromCallable()     | fromCallable()                                   |
| fromEvent(ev)        | RxBinding lib      |                    x                             |
| -----                | -----              | -----                                            |
| Subject              | Processor, Subject | Subject                                          |
| AsyncSubject         | AsyncSubject       |                    x                             |
| Subject              | PublishSubject     | PublishSubject                                   |
| BehaviorSubject      | BehaviorSubject    | BehaviorSubject                                  |
| ReplaySubject        | ReplaySubject      | ReplaySubject                                    |

# Creation
```dart
    Rx.range(3, 6) //=> 3, 4, 5, 6
        .listen((datum) => print("range: $datum"));

    // Note: If the second parameter('count') is not provided, then this will repeat forever
    Rx.repeat((index) => Stream.value(index), 4) //=> 0, 1, 2, 3
        .listen((datum) => print("repeat: $datum"));
```
# FromXXX

## fromFuture
Dart's `Future` is similar with JS's `Promise`. So we have `fromFuture` in RxDart as well: 

```dart
  Stream.fromFuture(myPromise())
      .listen((promisedValue) => print("fromPromise: $promisedValue"));
      //=> (3s) fromPromise: 23

  Future<int> myPromise() async {
    await Future.delayed(const Duration(seconds: 3));
    return 23;
  }        
```

## fromIterable
```dart
    Stream.fromIterable(data)
        .listen((datum) => print("item = $datum"));
    //=> item = 11, item = 19, item = 15
```

## Rx.fromCallable
This time the object we use is `Rx`, instead of `Stream`

```dart
  Rx.fromCallable(f1).listen(print); //=> Value1
  Rx.fromCallable(f2).listen(print); //=> (2s) Value2

  void f1() => "Value1";

  Future<String> f2() async {
    await Future<void>.delayed(const Duration(seconds: 2));
    return 'Value2';
  }
```

# Time

## timer
This time we don't use Stream, and use `Rx` instead

```dart
    print('start = ${DateTime.now().second}');
    Rx.timer("myValue", const Duration(seconds: 2))
        .listen((datum) => print("timeout: $datum, at ${DateTime.now().second}"));
    print('finish = ${DateTime.now().second}');
    //=> start, finish, (2s later) myValue
```

## interval
RxDart don't have a `Rx.interval()` method, because Stream already has such a method to do something periodly: 

```dart
    Stream.periodic(const Duration(seconds: 1), (num) => num + 10)
        .take(4)
        .listen((datum) {print('datum = $datum');}); //=> (1s)10, (2s)11, (3s)12, (4s)13
```


## RxJS's timer(delay, interval)
RxJS's `timer(time)` has a override function, which is `timer(delayTime, intervalTime)`. It will start working after the `delayTime`, and repeat the work every `intervalTime`. Yes, it's like an advanced version of `interval()`.

RxDart has no such function, but we can easily make such one by using `flatMap`.

```dart
    Rx.timer(0, const Duration(seconds: 1))
        .doOnData((event) => print("1s timeout "))
        .flatMap((value) => Stream.periodic(const Duration(seconds: 2), (num) => num))
        .take(2)
        .listen((datum) {print('datum = $datum');}); //=> (1s)time ends, (3s)0, (5s)1
```