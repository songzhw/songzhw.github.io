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

The subsequent code is my first version, which unfortunately is a awful code that will lead a ```StackOverflowError```.

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


When I type only one character, soon the app crashes! The error message is
![](/imgs/20151210_01.jpg)

Crash reason:
1. Type one character, app will map the OnTextChangeEvent to string, then EditText.setText()
2. Since EditText calls setText(), so the map() funciton is called again, which means EditText.setText() will be called again
3. .... (endless loop)
4. App is out of memory, then crashes!





## THe correct version
Since we have figured out the reason, we have solution: First filter the input, only the very string will be transmited to the subscriber()

If the input string is a normal integer, we will not to interfere the input.

But if the input string is a decimal number, we will filter the input.

The code is :
```kotlin
        WidgetObservable.text(etInputLimit)
            .filter{ input : OnTextChangeEvent ->
                var str = input.text().toString()
                var pointIndex = str.indexOf(".")
                pointIndex >=0
                    && str.substring(pointIndex).length > 3
            }
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

The previous code is a little superfluous to read. Then I refactor it. The final version is :
```kotlin
        WidgetObservable.text(etInputLimit)
            .map { it.text().toString() }
            .filter{ str : String ->
                var pointIndex = str.indexOf(".")
                pointIndex >=0
                    && str.substring(pointIndex).length > 3
            }
            .subscribe{ str : String ->
                var pointIndex = str.indexOf(".")
                if(pointIndex >= 0){
                    var ret = str.subSequence(0, pointIndex+3).toString()
                    etInputLimit.setText(ret)
                    etInputLimit.setSelection(ret.length)
                }
            }
```

In the previous final version, I reuse the map logic, and only call subscribe()  when the input is a decimal and has more than two decimal numbers.

Because of the filter condition, the etInputLimit.setText(ret) will not trigger subscriber() again. So the previous endless loop is avoid. 

For example, we type 0.864, and the steps are :
1. input 0 : map() --> filter() returns false;
2. input . : map() --> filter() returns false;
3. input 8 : map() --> filter() returns false;
4. input 6 : map() --> filter() returns false;
5. input 4 : map() --> filter() returns true; --> subscribte() calls 'et.setText("0.86")'
6. text is changed to '0.86' : map() --> filter() returns false;

Okay, now there is no endless loop.



## Another requirement
The another requirement is when you input "." as the first character, the EditText will turn the input into "0." automatically. How to resolve this problem?

Since this is another different requirement, which has different filter condition and map codes, we need to write another observable.
```kotlin
        WidgetObservable.text(etInputLimit)
            .map { it.text().toString() }
            .subscribe {
                if(it.startsWith(".")){
                    etInputLimit.setText("0$it")
                    etInputLimit.setSelection(it.length+1) //now has a extro "0", so +1
                }
            }

        WidgetObservable.text(etInputLimit)
            .map { it.text().toString() }
            .filter{ str : String ->
                var pointIndex = str.indexOf(".")
                pointIndex >=0
                    && str.substring(pointIndex).length > 3
            }
            .subscribe{ str : String ->
                var pointIndex = str.indexOf(".")
                if(pointIndex >= 0){
                    var ret = str.subSequence(0, pointIndex+3).toString()
                    etInputLimit.setText(ret)
                    etInputLimit.setSelection(ret.length)
                }
            }
```

**Note** : Two observable in one View is okay. When the condition is okay, two observable will be triggered in the same time!

## Conclusion

RxJava can be used in some similar situation, like :

http://stackoverflow.com/questions/12547801/unable-to-input-zero-after-decimal-point-in-android-edittext <br/>
Unable to input zero after decimal point in Android EditText

http://stackoverflow.com/questions/28734877/android-how-to-make-edittext-input-cant-be-started-with-zero0 <br/>
android how to make edittext input can't be started with zero(0)


RxJava is a switch knife in the Java world. I will introduce it, and apply it to the real Android developement in the following blogs.

If you have some thoughts , welcome to contact with me :  songzhw2012@gmail.com