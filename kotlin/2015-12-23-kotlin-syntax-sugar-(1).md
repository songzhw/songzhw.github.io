# Kotlin's Syntax Sugar

Honestly, Kotlin may be as powerful as Ruby or Groovy. However, writing Android project with Kotlin is also smooth and comfortable, thanks to multiple syntax sugar in kotlin.

This blog is actually from my sharing in my mobile group. Now I share it in my blog.

## 1. SAM Conversion
"SAM Type" is an interface or an abstarct class that has only one method.  This is can be  optimized by just use the funciton since Kotlin makes function as the first-class citizen.

Kotlin will transfer the SAM type into a lambda. And It will make your code more readable and simpler.
For example:

A. [java version]

```java
ivAd.setOnClickListener(new OnClickListener(){
	@Override
	public void onClick(View v){
		Toast.makeToast(actv, "str", Toast.LENGTH_SHORT).show();
	}
}
});
```

B. [kotlin version (SAM Conversion)]

```kotlin
ivAd.setOnClickListener{ v: View ->
	Toast.makeToast(actv, "str", Toast.LENGTH_SHORT).show();
}
```

## 2. lambda
Lambda will make your coding life much easier.

An obvious example is actually the hot "RxJava".

RxJava is hard to read, and the important logic codes are hidden in lots of SAM type and "{...}".
For example, I want to add all the png files in directories. I used Rxjava to stream these data.
A. java

```java
Observable.from(folders)
    .flatMap(new Func1<File, Observable<File>>() {
        @Override
        public Observable<File> call(File file) {
            return Observable.from(file.listFiles());
        }
    })
    .filter(new Func1<File, Boolean>() {
        @Override
        public Boolean call(File file) {
            return file.getName().endsWith(".png");
        }
    })
    .map(new Func1<File, Bitmap>() {
        @Override
        public Bitmap call(File file) {
            return getBitmapFromFile(file);
        }
    })
    .subscribeOn(Schedulers.io())
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(new Action1<Bitmap>() {
        @Override
        public void call(Bitmap bitmap) {
            imageCollectorView.addImage(bitmap);
        }
    });
```

B. kotlin

```kotlin
Observable.from(folders)
    .flatMap{ Observable.from(it.listFiles()) }
    .filter{ it.getName().endsWith(".png") }
    .map {getBitmapFromFile(it)}
    .subscribeOn(Schedulers.io())
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe{ imageCollectorView.addImage(bitmap)};
```

OMG, the same code in kotlin is much, much simpler, and readable.

To make our code right and simple, there are some points we have to know.
1. A normal lambda in Kotlin has the labmda arguments and the return type, like :

```kotlin
 view.setOnClickListener({ view -> toast("Click")})
```

2. We can get rid of the lambda arguments and return value, if they are not used.

```kotlin
view.setOnClickListener({ taost("Click"))
```

3. If the labmda has only one argument, then you can omit the argument declairation and use "it" keyword to represent the argument.
the "it" represents the argument : "view".

```kotlin
view.setOnClickListener({ taost("Click $it"))
```

4.  If the lambda is the last parameter in one function, we can move it out of the parentheses:
<br/>Take the form validation for an example. Only when the two EditText both has content, the commit button can be valid .

```kotlin
var et1Valid = WidgetObservable.text(etSearch)
    .map{ it. text().toString()}    // get the text of one EditText
var et2Valid = WidgetObservable.text(etInputLimit)
        .map{ it. text().toString()}  // get the text of another EditText

Observable.combineLatest(et1Valid, et2Valid){s1, s2 ->
    !TextUtils.isEmpty(s1) && !TextUtils.isEmpty(s2)
}.subscribe{
    btnCommit.isEnabled = it
}
```

(1). In the preceding code, I used RxAndroid v0.25. The *WidgetObservable* class is from RxAndroid.

(2). combineLatest() method is an example of mine. It takes three arguments, which are Observable1, Observable2, and Func2<T1, T2, R>

Func2 is a class that only has one method:
```R call(T1 t1, T2 t2)```

Since Func2 is a SAM type, so we can transfer it to lambda
And Because the Func2 argument is the last argument of combineLatest(), so we can abstract the Func2 out of combineLatest().


(3). combineLatest()<br/>
 :  Combines two source Observables by emitting an item that aggregates the latest values of each of the source Observables each time an item is received from either of the source Observables, where this aggregation is defined by a specified function.<br/>
You can find more detail about combineLatest() in :
http://reactivex.io/documentation/operators/combinelatest.html

When the content of either EditText changes, the combineLatest() will be triggered. This makes a good form validation.





## Reference
http://gank.io/post/560e15be2dca930e00da1083
"给 Android 开发者的 RxJava 详解"
http://antonioleiva.com/functional-programming-android-kotlin-lambdas/
"Unleash functional power on Android (I): Kotlin lambdas"