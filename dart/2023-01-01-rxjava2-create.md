# No more `Observable`
If you are familiar with RxJava or RxJS, you might be used to the use of `Observable` classes. However, don't be surprised if you can't find `Observable` in the RxDart package. That's because since RxDart 0.23.x, it remove all OBservable classes, and use the `Stream`'s extension instead. But no worries, this article, or even this series of articles, will introduce all the valuable details. 

# Create Observable
Rx world has two famous concept:
* `Observable`: create a stream to send out data or error(, also know as `upstream`)
* `Observer`: listen to a stream to receive data or error(, also know as `downstream`, `listener` or `subscriber`)

This article is mostly about how to create stream, or how to create Observable for us. 

## create Observable with several existing data

