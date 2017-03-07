Android data-binding framework has been invented more than one year, now I think it's time to research it and apply it to real project. So I will talk about the MVVM in this post. 

### RoboBinding
Before I talk about the Android data-binding framework, I want to talk about another data-binding library [RoboBinding](https://github.com/RoboBinding/RoboBinding). This library is created way before Android data-binding framework. I got to know this library two years ago, and found it very easy to use. This library uses the compile-time annotation to generate source code of data-binding for you. I have to say, this library is better than today(201703)'s Android data-binding library. Its two-way binding, AdapterView binding is even more easier than Android data-binding library. If you are interested, you can read the source code, and I hope you will enjoy the code.

### Data-Binding
You can open data-binding feature by using this:

```java
android { 
    ... ... ...
    dataBinding {
        enabled = true
    }
}
```

Then you can change the structure of layout xml, import <layout> and <data> and tell the layout xml you are using this data.

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android" >

    <data>
        <variable name="user" type="ca.six.bindingdemo.User"/>
    </data>

    <LinearLayout android:layout_width="match_parent" android:layout_height="match_parent"
        android:orientation="vertical">
        <TextView android:layout_width="wrap_content" android:layout_height="wrap_content"
            android:textSize="22dp" android:text="@{user.name}"/>
    </LinearLayout>

</layout>
```

And your Activity will be easy. No more "findViewById()", no more "tv.setText(...)", Pretty clean.

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    ActivityBindingDemoSimpleBinding binding = DataBindingUtil.setContentView(this, R.layout.activity_binding_demo_simple);

    User user = new User("name2", "desp2", true);
    binding.setUser(user);
}
```

Data-binding is not just that. There are more feature you need to learn:
* two-way binding
* custom attribute
* AdapterView's binding ; RecyclerView's binding
* ...

But this post is about the architecture, is about MVVM, so I will not talk too much about data-binding. I may create a new post about data-binding later.

### MVVM


### OneBindingAdapter


### Conclusion


