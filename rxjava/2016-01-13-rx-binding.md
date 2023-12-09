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
fun view.attachEvents() //Observable<T>: T is the event of view attaching, detaching
fun view.attaches() //Observable<Unit>
fun view.detaches() //Observable<Unit>

fun View.clicks() // Observable<Unit>
fun View.longClicks()  // Observable<Unit> 

fun View.focusChanges() // Observable<bool>

// :     view.viewTreeObserver .addOnDrawListener(listener)
fun View.draws() // Observable<Unit>
// :     view.viewTreeObserver .addOnGlobalLayoutListener(listener)
fun View.globalLayouts() // Observable<Unit>.  


// : view.setOnScrollChangeListener(listener)
fun View.scrollChangeEvents() // T has the data of : oldScrollX/Y, newScrollX/Y
```

# extensions on menu
```kotlin
// menuItem
fun menuItem.actionViewEvents()  // T is the event of menu collapsing or expanding
fun menuItem.clicks()  //Observable<Unit>

//: view.setOnDismissListener(listener)
fun PopupMenu.dismisses() // Observable<Unit>
//: view.setOnMenuItemClickListener(listener)
fun PopupMenu.itemClicks() // Observable<MenuItem>

//: view.setOnMenuItemClickListener(listener)
fun Toolbar.itemClicks()  // Observable<MenuItem> 
//: view.setNavigationOnClickListener(listener)
fun Toolbar.navigationClicks()  // Observable<Unit>
```

# extension on inputing views
```kotlin
// 实为: view.onItemClickListener = listener
fun AutoCompleteTextView.itemClickEvents()


// the 4 ext below are all based on:  view.addTextChangedListener(listener)
// T has the value of : String text, int start, int count, in after
fun TextView.beforeTextChangeEvents()
fun TextView.textChangeEvents() 
fun TextView.afterTextChangeEvents()
fun TextView.textChanges() // Observable<String>

//: view.setOnEditorActionListener(listener). 
//  action is the button at the bottom-right corner of IME, could be flexible, such as Send, Share, Return, Next, ...
fun TextView.editorActionEvents()  // T is actionId, such as: IME_ACTION_DONE, IME_ACTION_SEND, IME_ACTION_NEXT....
```

# extensions on interactiv views
```kotlin
//: view.setOnCheckedChangeListener(listener)
// CompoundButton's children: CheckBox, RadioButton, Chip, Switch, ToggleButton, ...
fun CompoundButton.checkedChanges() // Obserable<Bool>
//: view.setOnCheckedChangeListener {radioGroup, checkedId -> ... }
fun RadioGroup.checkedChanges() // Observable<Int>, Int is the checkedId


//the two ext below are both based on: view.setOnSeekBarChangeListener(listener)
fun SeekBar.changeEvents() // T includes: int progress, bool isFromUser
fun SeekBar.userChanges() // T includes: int progress
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