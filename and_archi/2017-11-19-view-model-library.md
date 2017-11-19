### 1. Why Do You Need Another Library: ViewModel?


#### 1.1 You need to handle the configuration change
The configuration change is totally out of your control. Last minute, your user is using your app. Next minute, he/she switch out and change a configuration(i.e. rotate the screen), your app may remove your current Activity and re-create it again. This is completely out of your control, but you have to deal with it. 

I know some developer would say, "It's okay for me. My project just make sure every Activity's orientation is portrait. So I have no such problems." But screen rotation is just one example of configuration change. You can make your Activity portrait, but you can forbit your user to change the language, or font size. Every time the user changed the language, like from English to Chinese, your activity will get notified this is a configuration change, and it may get removed and re-created. And you, as a developer, have to deal with it.

So, **The [ViewModel](https://developer.android.com/reference/android/arch/lifecycle/ViewModel.html) class allows data to survive configuration changes such as screen rotations**. 

#### 1.2 Why onSaveInstanceState() is not enough for us?
A tranditional way to handle the configuration change is to save data in the onSaveInstanceState() and to restore the data in onCreate(). 

But there are two limits about onSaveInstanceState(). 
1. onSaveInstanceState() can not save a large amounts of data. I've seen some posts that some developer saved a lot of data in the onSaveInstanceState(), and they got a `TransactionTooLargeException`.
2. The data you want to save in the onSaveInstanceState() must be serializable. So you'd better to use Parceable (Serializable is not a good option for Android applications). This is not just creating a class and making it implement Parceable. Sometimes the data is from third-party liarbry, and you can not modify the data. So for some scenarios, it's hard to save data in onSaveInstanceState().

### 2. A Simplest Demo










= = = = = = = = = = = = = = = = = = = = = = = = = = = = 
Why ViewModel is invented?
    *Configuration change. Not just rotate screen.
    *limit of onSavedInstanceState() : small, need serializable
Simple Sample
Questions about different instance of one Activity
    *How ViewModel is implemented
    *cannot replace onSaveInstanceState()
ViewModel vs. Static
    *sample code - config change
    *sample code - applicaion is terminated
    *difference
Caution
    *no context
    *need context, then use AndroidViewModel
