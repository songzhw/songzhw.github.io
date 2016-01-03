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

Espresso is a testing framework which is good at testing user interactions.

### Espresso 01 : Simple Example
A typical Espresso code is like this

```java

        onView(withId(R.id.btn_jumpform_jump))
                .perform(click())
                .check(matches(isDisplayed()));

```

* onView() is to find the view, it is like "findViewById".
* perfomr() is to take actions, like click, type text or something.
* check() is to verify the UI result is our expected result or not.

### Espresso 02 : Activity Jumping
If we are testing two Activity, Espresso is also help us test the UI easily.  For example, I have Activity A  and Activity B. 　I click the button in A, and jump to B with the text that is inputed in A.  After we call ``` (btn).perfom(click())```, should we wait the Activity B to start up? The answer is no. 

```java
    @Test
    public void szwJump_twoActv() {
        onView(withId(R.id.et_jumpfrom))
                .perform(typeText("hello"), closeSoftKeyboard());
        onView(withId(R.id.btn_jumpform_jump))
                .perform(click());

        // Clicking launches a new activity that shows the text entered above. You don't need to do
        // anything special to handle the activity transitions. Espresso takes care of waiting for the
        // new activity to be resumed and its view hierarchy to be laid out.
        onView(withId(R.id.tv_jumpto_display))
                .check(matches(withText("hello")));

        // going back to the previous activity
        pressBack();
        onView(withId(R.id.et_jumpfrom))
                .check(matches(withText( containsString("Espresso") )));
    }
```

Isn't it handy?

### Espresso 03 : ListView

```java

    @Test
    public void firstItemDisplay(){
        onView( withText("item: 0"))
                .check( matches(isDisplayed()));
    }

    @Test
    public void lastItemNotDisplay(){
        onView(withText("item: 99"))
                .check(doesNotExist());
    }


    // Espresso.onData(matcher) : Creates an DataInteraction for a data object displayed by the application.
    // DataInteraction : An interface to interact with data displayed in AdapterViews.
    // Matchers : http://hamcrest.org/JavaHamcrest/javadoc/1.3/org/hamcrest/Matchers.html
    private DataInteraction onRow(String str) {
        DataInteraction dataInteraction = onData(
                allOf(
                        instanceOf(ItemData.class),
                        MyAdapterMatcher.withName(str)
                )
        );

        return dataInteraction;
    }
```

onData() is like onView(). It's only used in AdapterView to find the items in AdapterView.

p.s : The preceding introduction is all about the Espresso-Core libray. For furture conditions, you can use Espresso-Intent library, or use Espresso-WebView library to test the Html UI and so on. 



## UiAutomator
Unlike Espresso, UiAutomator was design to test multiple apps, rather than one app. 

By now, Espresso is enough for my test requirement, so I am not familiar with UiAutomator actually.  If I use it later, I will expand this post about how to use UiAutomator better. 


## Conclusion
1. Monkey

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


2. Espresso

Espresso is a great tool for testing UI. I have use it into my project. I have to say Espresso sometimes will have weird bugs, like I call "``` perform(clicks())``` " , but the button is not clicked ; or Espresso will fail in one cellphone, but it will be okay in another cellphone.

In spite of these bugs, Espresso is actually a good UI test framework for most conditions. I recommend you to try it. 


3. test on NDK
with a single configuration in build.gradle, you can debug NDK codes

```groovy

```

But I don't have a efficient way to test NDK automatically. If I do, I will expand this post about it.
