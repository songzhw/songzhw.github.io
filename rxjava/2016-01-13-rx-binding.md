#RxBinding

## Click

```kotlin
    RxView.clicks(btnDoubleClick)
                .throttleFirst(2, TimeUnit.SECONDS)
                .subscribe{
                    println("szw click btnDoubleClick ! ${System.currentTimeMillis()}")
                }
```

**[long Click]**
```kotlin
    RxView.longClicks(view).subscribe(...);
```


## EditText.TextWatcher

```kotlin
        var et1Valid = RxTextView.textChangeEvents(etSearch)
            .map{ it. text().toString()}
```

## ListView
```kotlin
    RxAdapterView.itemClicks( listView )
    RxAdapterView.itemLongClicks( listView)
```
## Others

**[SnackBar]**
```kotlin
    RxSnackbar.dismisses(snackbar).subscribe(this::onSnackbarDismissed)
```