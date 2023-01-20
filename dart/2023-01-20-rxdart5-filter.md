Before we begin, we have two variables that you might need to know first. 
```dart
  final time500 = Duration(milliseconds: 500);
  final time1000 = Duration(seconds: 1);
```

## filter -> where
In the Dart, the filtering functionality of Stream is called `where`. Hence RxDart follow this naming pattern, so the famous `filter` operator (if you are familiar with RxJS or RxJava) has now called `where` in the RxDart. 

```dart
    Stream.periodic(time500, (idx) => idx)
        .where((event) => event % 2 == 1)
        .take(3)
        .listen(print); //=> 1, 3, 5
```        

## first, last
In the RxJS, `first(), last()` are all return a `Observable<T>`. And RxDart, unfortunately, has no such methods. It only has the `first`, `last` property which will return a `Future<T>` type. -- by the way, the `first`, `last` is not properties of RxDart, but the properties of Dart's Stream. 


```dart
  Future<void> first1() async {
    int firstElement = await Stream.periodic(time500, (idx)=>idx)
        .first; //=> 0 
    print('first = $firstElement');
  }
```

### how can I make a first operator
But is it possible to make a first or last operator, just like RxJS. Yes, it's possilbe, we just need to use `take(n)` or `takseLast(n)`. 

```dart
  final stream$ = Stream.periodic(time500, (idx)=>idx)
      .take(3);
  stream$.listen(print); //=> 0, 1, 2
```


```dart
  Stream<int> last5$ = Stream.periodic(time500, (idx)=>idx)
    .take(5) // will sent out 0,1,2,3,4
    .takeLast(2);
  last5$.listen(print); //=> 3, 4  
```

## take with conditions
There is `takeWhile`, `takeWhileInclusive`, `takeUntil` for us to choose. 

### takeWhile
`takeWhile(continueFn)` : as long as the continueFn return a true, then the stream will keep working. 

`takeWhileInclusive` will also sent out the first element that does not meet the continueFn. 

```dart
    Stream.periodic(time500, (idx)=>idx)
        .takeWhile((element) => element < 5)
        .listen(print); //=> 0, 1, 2, 3, 4

    Stream.periodic(time500, (idx) => idx)
        .takeWhileInclusive((element) => element < 5)
        .listen(print); //=> 0, 1, 2, 3, 4, 5        
```


### takeUntil
`ob1.takeUntil(ob2)` : ob2 is another stream. Once ob2 start to sent out data, then the ob1 will complete.

```dart
    Stream.periodic(time1000, (idx)=>idx)
        .takeUntil(Rx.timer("hi", Duration(seconds: 3)))
        .listen(print);  //=> 0, 1
```

Because at Second 1, the original stream sent out `0`
at Second 2, the original sent out `1`
at Second 3, ob2 start sending data, so our original ob1 will stop, and that's how we got the `0, 1` as the result. 