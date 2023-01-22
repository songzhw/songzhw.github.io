# No more `Observable`
If you are familiar with RxJava or RxJS, you might be used to the use of `Observable` classes. However, don't be surprised if you can't find `Observable` in the RxDart package. That's because since RxDart 0.23.x, it remove all OBservable classes, and use the `Stream`'s extension instead. But no worries, this article, or even this series of articles, will introduce all the valuable details. 

# Create Observable
Rx world has two famous concept:
* `Observable`: create a stream to send out data or error(, also know as `upstream`)
* `Observer`: listen to a stream to receive data or error(, also know as `downstream`, `listener` or `subscriber`)

This article is mostly about how to create stream, or how to create Observable for us. 

## cheat sheet
| RxJS                 | RxJava             | RxDart                                           |
|----------------------|--------------------|--------------------------------------------------|
|    x                 |        x           | StreamControler.stream                           |
| of(1,2,3)            | same as RxJS       | Stream.fromIterable([1,2,3])                     |
| range(start, count)  | same as RxJS       | Rx.range(start, end)                             |
| repeat(count)        | same as RxJS       | Rx.repeat( index=> Stream.value(index*2), count) |
| repeatWhen(ob2)      | same as RxJS       | x                                                |
| -----                | -----              | -----                                            |
| timer(time)          | same as RxJS       | Rx.timer(value, duration)                        |
| interval(time)       | same as RxJS       | Stream.periodic(duration, index => index)        |
| -----                | -----              | -----                                            |
| fromPromise(promise) | fromFuture(f)      | fromFuture(f)                                    |
| x                    | fromCallable()     | fromCallable()                                   |
| fromEvent(ev)        | RxBinding lib      | x                                                |
| -----                | -----              | -----                                            |
| Subject              | Processor, Subject | Subject                                          |
| AsyncSubject         | AsyncSubject       | x                                                |
| Subject              | PublishSubject     | PublishSubject                                   |
| BehaviorSubject      | BehaviorSubject    | BehaviorSubject                                  |
| ReplaySubject        | ReplaySubject      | ReplaySubject                                    |
