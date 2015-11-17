## Proguard in my kotlin app

### Release version has bugs?
My App use the library below :
```groovy
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    testCompile 'junit:junit:4.12'
    compile 'com.android.support:appcompat-v7:23.0.1'


    compile "org.jetbrains.kotlin:kotlin-stdlib:$kotlin_version"


    compile 'com.squareup.retrofit:retrofit:1.9.0'


    compile 'io.reactivex:rxjava:1.0.13'
    compile 'io.reactivex:rxandroid:0.25.0'


    compile 'com.nostra13.universalimageloader:universal-image-loader:1.9.4'


    compile 'com.journeyapps:zxing-android-embedded:3.0.3@aar'
    compile 'com.google.zxing:core:3.2.1'


    androidTestCompile 'com.android.support.test:runner:0.4'
    androidTestCompile 'com.android.support.test.espresso:espresso-core:2.2.1'


}
```

As you can see, I use the Kotlin, Retrofit, RxJava, RxAndroid, UIL, zxing.

When I try to build the app, I has encoutered many problems :
```
java.lang.ClassCastException: java.lang.Class cannot be cast to java.lang.reflect.ParameterizedType
```
<p>

```
java.lang.IllegalArgumentException: d.a: HTTP method annotation is required (e.g., @GET, @POST, etc.)
```

<p>

```
Parameter specified as no-null is null : method kotlin.jvm.internal.Intrinsics.checkParameterIsNotNull, parameter text
```
<p>
![](/imgs/20151117_01.jpg)    

These problems are only appeared in a release version of my app. So the Proguard is the number one suspect.

After a few hours, I have figured out the correct proguard configuration.

## My Proguard
```proguard

-ignorewarnings

-keepattributes Exceptions

# Retrofit
-keep class com.squareup.okhttp.** { *; }
-keep class retrofit.** { *; }
-keepclasseswithmembers class * {
    @retrofit.** *;
}
-keepclassmembers class * {
    @retrofit.** *;
}

# gson
##---------------Begin: proguard configuration for Gson  ----------
# Gson uses generic type information stored in a class file when working with fields. Proguard
# removes such information by default, so configure it to keep all of it.
-keepattributes Signature

# For using GSON @Expose annotation
-keepattributes *Annotation*

# Gson specific classes
-keep class sun.misc.Unsafe { *; }
#-keep class com.google.gson.stream.** { *; }

# Application classes that will be serialized/deserialized over Gson
-keep class cn.six.payx.entity.** { *; }

##---------------End: proguard configuration for Gson  ----------

# zxing
-keep class com.google.zxing.** {*;}
-keep class com.journeyapps.** {*;}

-keep class android.support.** { *; }
-keep class com.nostra13.universalimageloader.** {*;}
-keep class kotlin.** {*;}
-keep class rx.** {*;}

```