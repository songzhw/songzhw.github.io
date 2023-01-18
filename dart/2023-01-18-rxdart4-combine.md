This article is mostly about the operator to combine different observables. 

# operator of combiner

## Concat
`Rx.concat(ob1, ob2, ob2)` will listen to ob1 first. 
Once ob1 completes or goes wrong, then it will start to listen to ob2.
Once ob2 completes or goes wrong, then it complete itself.

```dart
Rx.concat([
  Stream.value(1),
  Rx.timer(2, Duration(seconds: 4)),
  Stream.value(3)
])
.listen(print); // prints 1, 2, 3
```
The keypoint here is the `3` is printed after 4 seconds. Because the third observable need to wait the second observable to be completed first. 


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
As we can see,  the first upstream actually could generate infinite data, and the second upstream only generate two data. 
But the result is our zip2 only generate two data, because once the second upstream completes, the whole zip2 would complete as well. 

Another note that you may noticed the data was printed every 500ms, and that's because the `zip` would wait all the upstreams generating data, and this would might be useful for you in some cases. 

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
    child: const Text("zip2耗时3秒还是5秒?")),
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



# high-order observable