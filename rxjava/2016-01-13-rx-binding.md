#RxBinding

## Click

```kotlin
        RxView.clicks(btnDoubleClick)
                .throttleFirst(2, TimeUnit.SECONDS)
                .subscribe{
                    println("szw click btnDoubleClick ! ${System.currentTimeMillis()}")
                }
```

## EditText.TextWatcher

```kotlin
        var et1Valid = RxTextView.textChangeEvents(etSearch)
            .map{ it. text().toString()}
```
