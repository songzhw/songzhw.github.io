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


# high-order observable