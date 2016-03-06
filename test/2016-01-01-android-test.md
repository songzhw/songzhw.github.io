# Test Android Project

Modern UI systems are always difficult to test. In a UI page, the user has dozens of choice to do different things, and the test aiming on the UI is really hard. 

I was told that iOS has a good test framework. It is like a camera. You can record what you did, and the framework will translate your action to a computer script. You only have to run these scripts later, and you can do you test job easily. 

Unfortunately, Android has no such thing. However, we can still do something about it. 

## JUnit
JUnit is also appliable for the Android project. It's can test the logic code, like database or something. It may not so useful for the UI test, and Internet has plenty of blogs about how to use JUnit. I want to focus on the UI test in this post, so I am not gonna talk about JUnit too much. 

p.s : When you create a new project, Android Studio already help you to create the test directory for you. 

![](/imgs/20160101_01.jpg)

* **test** directory : you can place your JUnit test codes here
* **androidTest** directory : you can place your Espresso test codes here


## Monkey

What is Moneky? Why is its name a animal?

It's because we assume that there are a naughty monkey that likes to touch the cellphone screen randomly. The "Monkey" test framework is like that.

It mocks a lot of pressing and touching, like a monkey. So the Monkey test framework are designed to do the presure test for your app.

Since monkey test framework is actually installed in every Android cellphone, so It will be easy to run the moneky test.

```shell
monkey -p cn.song.test --ignore-crashes 10000 > /mnt/sdcard/monkeyResult.txt
```

* **-p** : which package (or app) should we test?
* **--ignore-crashes** : Monkey will abort when it encounter a crash. If I do not want its abortion, use this tag
* **10000** : touch the screen 10,000 times 
* **> a.txt** : output the monkey test result to a.txt


## Espresso

This is a general post, so I am not go into too much deep about Espresso. 

Espresso is a white box-style test framework.  It's designed to test one app. The general three steps is :<br/>
(1). find the view by text or id<br/>
(2). do the action (like click or type text)<br/>
(3). confirm the result<br/>

One sample test is like this:

```java
import static android.support.test.espresso.Espresso.onView;
import static android.support.test.espresso.assertion.ViewAssertions.matches;
import static android.support.test.espresso.matcher.ViewMatchers.isDisplayed;
import static android.support.test.espresso.matcher.ViewMatchers.withText;


@RunWith(AndroidJUnit4.class)
@LargeTest
public class OneActivityTest{

    @Rule
    public ActivityTestRule<OneActivity> actvRule = new ActivityTestRule<>(OneActivity.class);

    @Test
    public void szwChangeText_sameActv() {
    	// type workds
        onView(withId(R.id.et_jumpfrom))
                .perform(typeText(str), closeSoftKeyboard());

        // click button
        onView(withId(R.id.btn_jumpform_reshow))
                .perform(click());

        // check the result
        onView(withId(R.id.tv_jumpfrom_display))
                .check(matches(withText(str)));
    }

}
```

In this example, you type some words in a EditText, close the soft keyboard, then click a Button. My expection is my TextView shows the newly typed words.


p.s : The preceding introduction is all about the Espresso-Core libray. For furture conditions, you can use Espresso-Intent library, or use Espresso-WebView library to test the Html UI and so on. 



## UiAutomator
Unlike Espresso, UI Automator is more like a black box-style test framework. Beside, UiAutomator can be used to test multiple apps, rather than one app.

Actually, UI Automator is the same three steps like Espresso. However, since UI Automator is a black box-style, the QA definitly do not know the ID of one View. In this case, QA can use the **UI Automator Viewer** tool to help them find the important information about one view.

**UI Automator Viewer** lies in the Android SDK, and its precise location is ``` %android_sdk% / tools/ uiautomatorviewer.bat (windows OS）```. 

Of course, I am not a programmer in Quora. But I can use **UI Automator Viewer** to analyze Quora app's information.

![](/imgs/20160306_01.jpg)

At the first, I thought the question list in Quora app is a ListView. But when using this tool, I found out it turns out a WebView.

If you are a QA, you can use this tool to get app's information too. 

Here is a example, clicking a button , then jumping to another Activity with text for argument. 

```java
@RunWith(AndroidJUnit4.class)
public class JumpActivitiesTest extends BlackTest {

    @Test
    public void testJump() throws UiObjectNotFoundException {
        UiAutoUtil.openMyApp(device);

        UiObject et = device.findObject(new UiSelector().className("android.widget.EditText"));
        et.setText("Tomorrow");

        UiObject btnJump = device.findObject(new UiSelector().text("Open activity and change text"));
        btnJump.clickAndWaitForNewWindow();

        UiObject tv2 = device.findObject(new UiSelector().resourceId("cn.six.aut:id/tv_jumpto_display"));
        Assert.assertEquals("Tomorrow", tv2.getText());
    }

}
```


## Conclusion
#### 1. Monkey

Monkey sometimes will crash in a weired way. 
For example. A single thread situation :

```java
if(tvName != null){
	tvName.setText("error");
}
```

but monkey may crash in the "tvName.setText("error");" line with the NullPointerException.

Since it is a single thread situation, so tvName of course is not null, but the Monkey will report a NPE, which is really confusing.

So Monkey may have some good points, but I am not a fan of Monkey test. And reading the monkey test result to recreate the crash situation is really torture, so fix the crash bug is an arduous work.


#### 2. Espresso

Espresso is a great tool for testing UI. I have use it into my project. I have to say Espresso sometimes will have weird bugs, like I call "``` perform(clicks())``` " , but the button is not clicked ; or Espresso will fail in one cellphone, but it will be okay in another cellphone.

In spite of these bugs, Espresso is actually a good UI test framework for most conditions. I recommend you to try it. 

#### 3. UI Automator

UI Automator is a powerful black box-style tools. And so is UI Automator Viewer. I will talk about it later more.

#### 4. test on NDK

with a single configuration in build.gradle, you can debug NDK codes

```groovy
    buildTypes {
        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
        debug {
            jniDebuggable true
        }
    }
```

But I don't have a efficient way to test NDK automatically. If I do, I will expand this post about it.
