## 1. Why Do You Need Another Library: ViewModel?

### 1.1 You need to handle the configuration change
The configuration change is totally out of your control. Last minute, your user is using your app. Next minute, he/she switch out and change a configuration(i.e. rotate the screen), your app may remove your current Activity and re-create it again. This is completely out of your control, but you have to deal with it. 

I know some developer would say, "It's okay for me. My project just make sure every Activity's orientation is portrait. So I have no such problems." But screen rotation is just one example of configuration change. You can make your Activity portrait, but you can forbit your user to change the language, or font size. Every time the user changed the language, like from English to Chinese, your activity will get notified this is a configuration change, and it may get removed and re-created. And you, as a developer, have to deal with it.

So, **The [ViewModel](https://developer.android.com/reference/android/arch/lifecycle/ViewModel.html) class allows data to survive configuration changes such as screen rotations**. 

### 1.2 Why onSaveInstanceState() is not enough for us?
A tranditional way to handle the configuration change is to save data in the onSaveInstanceState() and to restore the data in onCreate(). 

But there are two limits about onSaveInstanceState(). 
1. onSaveInstanceState() can not save a large amounts of data. I've seen some posts that some developer saved a lot of data in the onSaveInstanceState(), and they got a `TransactionTooLargeException`.
2. The data you want to save in the onSaveInstanceState() must be serializable. So you'd better to use Parceable (Serializable is not a good option for Android applications). This is not just creating a class and making it implement Parceable. Sometimes the data is from third-party liarbry, and you can not modify the data. So for some scenarios, it's hard to save data in onSaveInstanceState().

## 2. A Simplest Demo
**Step 01** - Create your ViewModel class
Just make it extend ViewModel. 
```java
public class ZeroViewModel extends ViewModel {
    public User user;
}
```

**Step 02** - use it in the Fragment/FragmentActivity
```java

public class ZeroDemo extends AppCompatActivity {
    private TextView tv;
    private ZeroViewModel vm;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_tv_btn);
        tv = findViewById(R.id.tv_simple);
        
        vm = ViewModelProviders.of(this).get(ZeroViewModel.class);
        System.out.println("szw vm.user = " + vm.user);
    }
    
    public void onClickSimpleButton(View v) {
        vm.user = new User(23, "jorden");
    }
}
```
Note that `ViewModelProvides.of()` needs a Fragment or a FragmentActivity as a parameter. 
By the way, AppCompatActivity is a subclass of Fragment, so you can pass an AppCompatActivity object here.

You can rotate screen now, and you will find out the value of `vm.user` is always there. That's what the ViewModel's for.


## 3. Different Instance of One Activity Class

### 3.1 Having two instance at the same time
Let's do an experiment. Here is the code.

```java
public class SameClass01 extends AppCompatActivity {
    private TextView tv;
    private ZeroViewModel vm;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_tv_btn);
        tv = findViewById(R.id.tv_simple);

        vm = ViewModelProviders.of(this).get(ZeroViewModel.class);
        System.out.println("szw SameClass01 : " + vm.user);

    }
    // launch the second instance
    public void onClickSimpleButton(View v) {
        vm.user = new User(100, "SuperMario");
        startActivity(new Intent(this, SameClass01.class));
    }
}
```

And when the second launch is created, the log is :
`I/System.out: szw SameClass01 : null`

So even you have two instance of one Activity, the ViewModels is not a mess. The ViewModel in the first instance holds the "Mario" user, and the ViewModel in the second instance holds the null user. 

### 3.2 Creating the second instance later
Now if I open SameClass02 (an Activity), save one user onto ViewModel, then exit SameClass02. 
Later, I open SameClass02 again, what value would I get from the ViewModel's user.

```java
public class SameClass02 extends AppCompatActivity {
    private TextView tv;
    private ZeroViewModel vm;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_tv_btn);
        tv = findViewById(R.id.tv_simple);
        vm = ViewModelProviders.of(this).get(ZeroViewModel.class);
        System.out.println("szw SameClass02 onCreate() : " + vm.user);
    }

    public void onClickSimpleButton(View v) {
        vm.user = new User(22, "test");
    }


    public void onClickSimpleButton2(View v) {
        System.out.println("szw SameClass02 : saved = "+vm.user);
    }
}

```

From the log, `System.out: szw SameClass02 onCreate() : null`, we are glad to see the ViewModel is not messed.

## 4. 


















= = = = = = = = = = = = = = = = = = = = = = = = = = = = 
ViewModel vs. Static
    *sample code - config change
    *sample code - applicaion is terminated
    *difference
Caution
    *no context
    *need context, then use AndroidViewModel
