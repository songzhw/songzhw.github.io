Since RxBinding v4.0+, the old approach is not existing anymore, which means the code below are non-exist in the RxBinding v4.0+ package. 

```kotlin
RxView.clicks(btn)
RxView.longClicks(view)
RxTextView.textChangeEvent(et)
RxAdapterView.itemClicks(lv)
RxAdapterView.itemLongClicks(lv)
RxSnackbar.dismiss(snackbar)
... ...
...
```


RxBinding 4.0+ is taking advantage of Kotlin extension method, and send out an observable like `view.clicks()`. 

Since there is no official doc to introduce all the extension methods of RxBinding. So this topic will cover the most common extension methods you might need. 

# extensions on View
```kotlin

```

# extensions on menu
```kotlin

```

# extensions on interactiv views
```kotlin

```

# extensions in the independent packages
You can import the extensions from these packages:
```gradle
implementation 'com.jakewharton.rxbinding4:rxbinding-drawerlayout:4.0.0'
implementation 'com.jakewharton.rxbinding4:rxbinding-swiperefreshlayout:4.0.0'
implementation 'com.jakewharton.rxbinding4:rxbinding-recyclerview:4.0.0'
```

And extension methods are: 
```kotlin
// : view.addDrawerListener(listener)
fun DrawerLayout.drawerOpen() // Observable<bool>, bool isOpen

// : view.setOnRefreshListener(listener)
fun SwipeRefreshLayout.refreshes()  //Observable<Unit>
```

## extensions on RecyclerView
```kotlin
import androidx.recyclerview.widget.RecyclerView.Adapter

// : adapter.registerAdapterDataObserver(listener.dataObserver)
fun Adapter.dataChanges() // Observable<Adapter>

// : rv.addOnChildAttachStateChangeListener(listener). 
fun RecyclerView.childAttachStateChangeEvents()

// : rv.onFlingListener = listener
fun RecyclerView.flingEvents()

// : rv.addOnScrollListener(listener)
fun RecyclerView.scrollStateChanges() //scroll state: idle, settling, scrolling
fun RecyclerView.scrollEvents()  //scroll data: dx, dy
```