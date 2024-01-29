This article is mostly about the operator to combine different observables. 

# operator of combiner

## Concat
* `Rx.concat(ob1, ob2, ob2)` will listen to ob1 first. 
* Once ob1 completes or goes wrong, then it will start to listen to ob2.
* Once ob2 completes or goes wrong, then it complete itself.

```dart
Rx.concat([
  Stream.value(1),
  Rx.timer(2, Duration(seconds: 4)),
  Stream.value(3)
])
.listen(print); // prints 1, 2, 3
```
The keypoint here is the `3` is printed 4 seconds after `2` is printed. Because the third observable need to wait the second observable to be completed first. 


Another approach to use concat is the `concatWith` operator: 
```dart
stream1.concatWith(stream2).listen(print);
```

### error on concat
What if the upper stream went wrong, what would happen?
```dart
Stream.value(10)
  .concatWith([
    Stream.error(Exception("error12")),
    Rx.timer(15, const Duration(seconds: 2)),
  ])
  .listen(print); //=> 10, "Unhandled Exception: error12", (2 seconds later) 15
```

### Three-level cache
```
Rx.concat(memoryCache(), databaseCache(), endpointCache())
  .take(1) // only take the first cache
  .listen(print) //=> take the first elgible cache
```
p.s. Unlike RxJava, RxDart does not have the `first` or `firstElement` operator, so we have to use `take(1)` to do the same job here.

## zip
Rx actually has the `zip2, zip3, zip4, ..., zip9` methods. Take `zip2` for example, it takes in two observables/streams, and combine their result one by one as a result. 

Note that once any upstream completes, the zip would complete as well. 
```dart
Rx.zip2(
  Stream.periodic(const Duration(milliseconds: 500), (value) => value),
  Stream.fromIterable(["x", "y"]),
  (a, b) => "($a, $b)"
)
  .listen(print); //=> (0,x), (1,y)
```

### two details
1). As we can see,  the first upstream actually could generate infinite count of data, and the second upstream only generate two data. 

But the result is our zip2 only generate two data, because once the second upstream completes, the whole zip2 would complete as well. 

2). Another note that you may noticed the data was printed every 500ms, and that's because the `zip` would wait all the upstreams generating data, and this would might be useful for you in some cases. 

### is upstreams in parallel?
```dart
OutlinedButton(
    onPressed: () {
      print('start = ${DateTime.now().second}'); //=> start(52s)
      Rx.zip2(
        Rx.timer(11, const Duration(seconds: 2)), 
        Rx.timer(25, const Duration(seconds: 3)),
          (a, b) => a + b
      )
        .listen((datum) => print("got $datum at ${DateTime.now().second}")); //=> got 36(55s)
      print('end = ${DateTime.now().second}'); //=> end(52s)
    },
    child: const Text("zip takes 3 seconds or 5 seconds?")),
```
As you can see, since the data is printed at Second 55, so it means the whole zip2 just takes 3 seconds, rather than 3 + 2 = 5 secons. From this experiment, we know all the upstreams of zip would be run in parallel. 


## Merge
Merge is simple, `merge(ob1, ob2).listen()` would mean every data of ob1 and every data of ob2 would go through the listen method. 

```dart
Rx.merge([
  Stream.periodic(const Duration(seconds: 1), (idx) => idx)
    .map((d) => "A$d")
    .take(2),
  Stream.periodic(const Duration(milliseconds: 400), (idx) => idx)
    .map((d) => "B$d")
    .take(4),
]).listen(print); //=> B0, B1, A0, B2, B3, A1
```

# combineLatest
In RxDart, the method of combining latest stream is `combineLatest2, combineLatest3, ...`

```dart
    final wind = Stream.periodic(twoSec, (idx) => idx).map((d) => "A$d").take(2);
    final temperature = Stream.periodic(ms400, (idx) => idx)
        .map((d) => "B$d")
        .take(4);

    Rx.combineLatest2(wind, temperature, (w, t) => [w, t])
        .listen(print)
        .clearBy(disposables); //=> (2s)[A0, B3], (4s)[A1, B3]
    //combineLatest, just like zip, will wait all upstream has a value then notify the subscriber
```

#  withLatestFrom
* CombineLatest will trigger once any upstream get updated
* WithLatestFrom, such as `a.withLatestFrom(b)`, will trigger when the `a` is updated.

```dart
    final wind = Stream.periodic(oneSec, (idx) => idx).map((d) => "w$d").take(2);
    final temperature = Stream.periodic(ms400, (idx) => idx).map((d) => "t$d").take(6);

    temperature // temperature is the trigger !
        .withLatestFrom(wind, (t, w) => "t$t-w$w")
        .listen(print)
        .clearBy(disposables); //=> t2-w0, t3-w0, t4-w1, t5-w1
```

# race
Many streams, the first one that sends out a data will win the race. Others upstream will get canceled.

```dart
    final s1 = Rx.timer("A", threeSec)
        .doOnDone(() => print("A done"))
        .doOnCancel(() => print("A cancelled"));
    final s2 = Rx.timer("B", twoSec)
        .doOnDone(() => print("B done"))
        .doOnCancel(() => print("B cancelled"));
    Rx.race([s1, s2]).listen(print).clearBy(disposables);
    //=> A cancelled; B, B done, B cancelled
```

# forkJoin
ForkJoin will send out the result array, just like zip. But forkJoin will only send out the last data of every upstream:

```dart
    print('szw start nows: ${DateTime.now().second}');
    final s1 = Stream.periodic(oneSec, (idx) => "A$idx").take(4);
    final s2 = Stream.periodic(oneSec, (idx) => "B$idx").take(5);
    Rx.forkJoin2(s1, s2, (a, b) => "[$a, $b]")
        .listen((ev) => print('szw end $ev nows: ${DateTime.now().second}'))
        .clearBy(disposables); //=> starts (50s), ends [A3, B4] (55s)
```

# pairwise
This will keep record of the previous data, and combine it with the current data, to make a pair. 

```dart
    Stream.periodic(oneSec, (idx) => idx)
        .take(4)
        .pairwise()
        .listen(print)
        .clearBy(disposables);
    //=> [0, 1], [1, 2], [2, 3]
```

This operator is useful when you have to calculate touch events. You will need the previous touchEvent data to calculate the offerX or offerY.

```dart
touch$.pairwise()
  .map( (e1, e2) => e2.x - e1.x)
  .listen(dx => ....)
```


# high-order observables
Unfortunately, `rxdart: ^0.27.7` still has not support the high-order observable very well. High-order observable means the observables that emit not the data T, but emit the Observable<T>. 

And all the `concatAll`, `mergeAll`, `zipAll`, `combineAll`, `switchAll`, `exhaust` operators that exist in RxJS, are all missing in the RxDart.  Hope the RxDart team will bring them into the RxDart world in the future. 

The only high-order operator I can show here is `switchLatest`.
```dart
    Rx.switchLatest(
      Stream.fromIterable(<Stream<String>>[
        Rx.timer('A', const Duration(seconds: 2)),
        Rx.timer('B', const Duration(seconds: 1)),
        Stream.value('C'),
      ]),
    ).listen(print).clearBy(disposables); //=> C
```    

Note that the switchLatest's data is no longer string, but the `Stream<String>`. And it will only emit most-recently-emitted data, which is C in the previous example. 