This post is a guidance of how to replace some specific views globally. By globally, I mean I need to replace all the views that meet our needs. 

### AppCompatEditText
Before diving into the ocean of details, I want to talk about one relevant View: AppCompatEditText.

AppCompat***View is designed for the UI on the older versions of the platform. Then your Android 4.2 may have the Lollipop UI.  The picture below can explain this to you.

![](./_image/2017-10-10-20-13-58.jpg)


[The document of AppCompatEditText](https://developer.android.com/reference/android/support/v7/widget/AppCompatEditText.html) already tell us :
```
This will automatically be used when you use EditText in your layouts and the top-level activity / dialog is provided by appcompat.
You should only need to manually use this class when writing custom views.
```

The second line actually are suggesting you to use AppCompatEditText, not EditText, as a parent of custom view. If you insist to use EidtText, you will get a Lint warning.

![](./_image/2017-10-09-19-10-43.jpg)

The first line is the essential of this post. "This will automatically be used when you use EditText in your layout". What? How? How do you achieve this? 

This means even you write <EditText> in the layout xml files, and you will get a <AppCompatEditText> eventually. This is what we want to do too, replacing views globally.




p.s. Some pictures are from :
[http://www.jianshu.com/p/4c2d90d850df](http://www.jianshu.com/p/4c2d90d850df)

