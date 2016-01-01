# Test Android Project

Modern UI systems are always difficult to test. In a UI page, the user has dozens of choice to do different things, and the test aiming on the UI is really hard. 

I was told that iOS has a good test framework. It is like a camera. You can record what you did, and the framework will translate your action to a computer script. You only have to run these scripts later, and you can do you test job easily. 

Unfortunately, Android has no such thing. However, we can still do something about it. 

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
