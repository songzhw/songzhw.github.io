---
layout: post
category : Android
tagline: "kotlin"
tags : [kotlin]
---

## A few steps to use NDK in Kotlin

First of all, I have to mention the version of Kotlin, NDK, AndoridStudio.  Because it seems not as same as the old versions.

<table class="table table-bordered table-striped table-condensed">
    <tr>
        <td> NDK </td>
        <td> Kotlin </td>
        <td> Android Studio </td>
    </tr>
    <tr>
        <td> android-ndk-r10e </td>
        <td> 1.0.0-beta-1103 </td>
        <td> 14.1 </td>
    </tr>
</table>

#### Step 1: modify [local.properties] in the project root directory

add a new line  (windows path is a littler different, please be careful)

{% highlight %}
ndk.dir=E\:\\android-ndk-r10e
{% endhighlight %}


#### Step 2: modify [gradle.properties] in the project root directory

add a new line

{% highlight %}
android.useDeprecatedNdk=true
{% endhighlight %}

#### Step 3:  modify [build.gradle] in the app module or the modules which are using NDK

add the ndk config in the android.defaultConfig path.

{% highlight %}
android {
    defaultConfig {
	    // ....

        ndk{
            moduleName "firstNdk"
        }

    }
}
{% highlight %}

#### Step 4: create a java class that is using JNI

![](/imgs/20151114_01 "build JniUtil.java here")


{% highlight java %}
package cn.six.payx.util;

public class JniUtil {

    public static native String getKey();
    public native int getNo();

    static{
        System.loadLibrary("firstNdk");
    }


}

{% endhighlight %}


**Note**: Yes, I use Java to conmunicate with C, **instead of Kotlin**.<br/>
Because Kotlin does not support the static memeber/function in Java, and native anntaion is changed in Kotlin 1.0.0 beta.
Now I have no clue about how to write JNI in Kotlin.  So I use Java.<br/>
Fortunately, Kotlin can use Java classes pretty easy.

#### Step 5:


#### Step 6:


#### Step 7:


#### Step 8:



### Source of this sample
The sample is in my "CleanBeta-Kotlin" prject, that is here :<br/>
<a href="https://github.com/songzhw/CleanBeta-Kotlin">https://github.com/songzhw/CleanBeta-Kotlin</a>

{% highlight java %}

{% endhighlight %}


