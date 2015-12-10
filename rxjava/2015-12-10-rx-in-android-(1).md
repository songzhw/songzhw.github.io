# Limit to EditText's input

## introduction

In this blog, all codes are write by Kotlin and RxJava <br/>
(I used RxAndroid v0.25 just for convenience.
If you like, you can change it to Jake Wharton's Rxbinding library and RxAndroid v1.0.)


We now have a EditText to input a number. And we want the EditText:
1. only showing numbers with 2 decimals at all times
2. If the first character is ".", EditText automatically becomes "0."


## My first and wrong version

Our steps are of course listen to the input of a EditText, then swift the data, control the showing text.

So I have map(), filter() in my mind.

The subsequent code is my first version, which unfortunately is a awful code that will lead a ```StackOverflowError```

```kotlin
        WidgetObservable.text(etInputLimit)
            .map { input : OnTextChangeEvent ->
                // CharSequence -> String
                var str = input.text().toString()

                // get the number
                var ret = str
                var pointIndex = str.indexOf(".")
                if(pointIndex >= 0){
                    ret = str.subSequence(0, pointIndex+3).toString()
                }
                ret
            }
            .subscribe{ str : String ->
                println("szw $str")
                etInputLimit.setText(str)
                etInputLimit.setSelection(str.length)
            }
```



## THe correct version

## Another requirement


## Conclusion