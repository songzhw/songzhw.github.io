# Two set of similar RxJava operations

 * throttleFirst() vs. debounce()
 * combineLatest() vs. zip()

## combineLatest() vs. zip()

### Form Validation

Now I have two EditText for userName and password and a button to commit the previous two string. When either EditText is empty, the button should be invalide (cannot be clicked). 

normal thought
```java
etName.addTextWatcher{
	if(!TextUtils.isEmpty(str)) { isValid01 = true;}
	if(isValid01 && isValid02) { 
		btn.setEnable(true);
	}
	else {
		btn.setEnable(false);
	}
}
etPwd.addTextWatcher{
	if(!TextUtils.isEmpty(str)) { isValid02 = true;}
		if(isValid01 && isValid02) { 
		btn.setEnable(true);
	}
	else {
		btn.setEnable(false);
	}
}
```

The new code with RxJava( and Kotlin) is :
```kotlin
   fun validForm(){
        var et1Valid = WidgetObservable.text(etSearch)
            .map{ it. text().toString()}
        var et2Valid = WidgetObservable.text(etInputLimit)
                .map{ it. text().toString()}

        Observable.combineLatest(et1Valid, et2Valid){s1, s2 ->
            !TextUtils.isEmpty(s1) && !TextUtils.isEmpty(s2)
        }.subscribe{
            btnForm01.isEnabled = it
        }
    }
```

In this way, if either EditText's input changes, combineLatest() will be called. 

** combineLatest(Observable<T1>, Observable<T2>, Fun<T1, T2, R> ) ** : Merges two observable sequences into one observable sequence by using the selector function whenever one of the observable sequences produces an element.

note: in the rxjava world, "T1, T2, ..." means the argument type<br/>
"R" means the return value type.

note: the kotlin block shoule not use the "return" keyword. And the last code in a block is actually the returned vaue. Groovy and ruby has the same feature.


### defference between throttleFirst() and zip()

zip() also as the "Observable<T1>, Observable<T2>, Fun<T1, T2, R> " argument like combineLatest(). However zip will not call subscribe() till all Observable are finished. 

combineLatest() is not like this. When Observable<T1> or Observable<T2>, either of them , is changed, subscribe() is called. In some way, combineLatest is more silimar to merge(), rather than zip()



## throttleFirst() vs. debounce()


```kotlin 
         ViewObservable.clicks(btnDoubleClick)
                 .throttleFirst(2, TimeUnit.SECONDS)
                 .subscribe{		                
					println("szw click btnDoubleClick ! ${System.currentTimeMillis()}")
                 }
```

** throttleFirst ** : 





## Reference
