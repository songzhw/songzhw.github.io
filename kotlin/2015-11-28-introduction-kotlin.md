# Welcome to the Kotlin world

We already talked the multiple problems in Android development in the second blog : "2015.11.17 Clean App (2) : Boilerplate in Android".

From then on, we fix the boilerplate problem in the "clean app(2)" blog, and fix the God Activity class and MVC framework problem in the "clean app(3) / (4) / (5)". Now the unresolved problems are only:
* Complex Multi-Thread Handling 
* (Java) file stream, try-catch, no block, no first-class function, no AOP ....

In this blog and maybe later multiple blogs, we will try to fix the last problem : the java language that maybe a little clumsy nowadays. And my final choose is **Kotlin**


## Comparison

Firstly, I'd like to choose other languages to code Android. Right now, I have some candidates:

### groovy
Since I am a fan of Ruby, and Groovy is similar to Ruby.
So I have to say, Groovy is really my favorite candidate.

Groovy has release its 2.4 version in the early 2015. The biggest feature of Groovy v2.4 is : Groovy support Android now!

I was so excited. Image the closure, lambda, file operation, metaprogramming and other dozens of syntax sugar, all appear in the Andorid application.
 ^-^, How beautiful is that!

However, when I really write a simple HelloWorld android application with Groovy, I start to realize groovy may be not the perfect language right now.


The biggest reason is : Groovy has too many methods!

A simple HellowWorld project created by AndroidStudio actually already exceed 40,000 methods.<br/>
If I also use RxJava, Retrofit, OkHttp, zxing and so on, Oh my god, the project already exceed 50,000 methods, which is very closed to 65536 limit !
```
Groovy            --  30,656 methods
support.v7        --  12,319 methods
rxjava,rxandorid  --  nearly 4200 methods
zxing             --  neraly 2500 methods
retrofit,gson     --  neary 1600 methods
...
```

Besides, even we can use multiple dex technology to expand the 65536 limit, however, the multidex application has a slower startup speed, a more complex structure. <br/>
So, I believe, a short and concise application is simpler and more beautiful. What I should do is to make my application short, or, in another word, has less methods!

**Conclusion** : Even if I really love Groovy, but I have to say the current version of Groovy for Android is too heavy to write an Android application.



### kotlin

I start to know Kotlin by chance because Jake Wharton recommended it. When I write an Android Applicaiton with Kotlin, I start to accept Kotlin.

Kotlin has some adavantages:
* 7K+ method count
* many syntax sugar
* compatible with Java codes (in most ways)
* Kotlin Extension for Andorid is really handy
<p><p>
Note:
1. Kotlin v0.13 has 7K+ methods<br/>
Kotlin v0.14 increased to 10K+<br/>
However, kotlin v1.0-Beta has return to 7K+.<br/>

Clearly, the Kotlin producer knows what the method count means in the Android application, so the v1.0-beta deleted nearly 3K methods.

2. Kotlin is full of syntax sugar, compared to Java. I will introduce them and apply them to the Android application in future.

3. Kotlin can use Java directly, which means our android project can have java and kotlin together. This is really convenient to reuse the old java codes. <br/>
However, static field/method, generics ... in Java is different from kotlin.  I will talk about them too later.




## Introduction to Kotlin
Kotlin is a statically-typed programming language that runs on the Java Virtual Machine and also can be compiled to JavaScript source code.
 Its primary development is from a team of JetBrains programmers.

Development lead Andrey Breslav has said that Kotlin is designed to be an industrial-strength object-oriented language, and to be a better language than Java but still be fully interoperable with Java code, allowing companies to make a gradual migration from Java to Kotlin.

I will not introduce the specific syntax. You can learn Kotlin throught this website:
https://kotlinlang.org/docs/reference/basic-syntax.html

And this is the website to tell you how to begin write Android applications with Kotlin:
http://antonioleiva.com/kotlin-for-android-introduction/
http://antonioleiva.com/kotlin-android-create-project/
http://antonioleiva.com/kotlin-android-extension-functions/



But I will keep writing blogs about how to apply Kotlin in Android. 



## Later Blogs
* How to use Kotlin in Android
* Kotlin trap
* experience of apply kotlin to Android projects
* Kotlin notes : NDK
* Kotlin notes : release package
* Kotlin notes : 


