This post is a guidance of how to replace some specific views globally. By globally, I mean I need to replace all the views that meet our needs. 

### AppCompatEditText
Before diving into the ocean of details, I want to talk about one relevant View: AppCompatEditText.

AppCompat***View is designed for the UI on the older versions of the platform. Then your Android 4.2 may have the Lollipop UI.  The picture below can explain this to you.

![Difference between EditTet and AppCompatEditText](http://upload-images.jianshu.io/upload_images/2833342-3e0976fadc11aef7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

[The document of AppCompatEditText](https://developer.android.com/reference/android/support/v7/widget/AppCompatEditText.html) already tell us :
```
This will automatically be used when you use EditText in your layouts and the top-level activity / dialog is provided by appcompat.
You should only need to manually use this class when writing custom views.
```

