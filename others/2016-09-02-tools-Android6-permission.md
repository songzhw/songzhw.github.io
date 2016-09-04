Since the Android 6.0 is published, the dynamic permission request system is a good news for the user. However, it means we developer will code more, even more biolerplate. 

Is there a better way to code more easier? Yes, `Permission6` is here to rescue us.

Before I introduce the `Permission6` to you, I have to say there are many libraries to do such a job. Most of them requires you to subclass your Activity to their Activity. This way, their Activity can do the `Activity.onRequestPermissionsResult()` and `Activity.requestPermissions()` job for you. 

However we ususaly have our own base Activity as a parent for all our own Activity, and Java only allows sinlge inheritance, so these libraries I mentioned before can't meet our requirement. 

That's why Permission6 is different. Permission6 library uses the  `Favor composition over inheritance` principle. You don't have to declair your Activity as an extension of some specific Activity!

#### import Permission6

```groovy
  dependencies {
    compile 'ca.six.util:Permisssion6:1.0.1'
  }
```

#### How to use Permission6
If you want to apply the "WRITE_EXTERNAL_STORAGE" permissin, you can use `Permission6.executeWithPermission()` to request the permission, and use `IAfterDo` interface to deal with the result.

Here is the code:

```java
public class MainActivity extends AppCompatActivity implements View.OnClickListener, IAfterDo {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        findViewById(R.id.btn_main).setOnClickListener(this);
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        Permission6.destory();
    }

    // ========== request the permission ========== 
    @Override
    public void onClick(View view) {
        Permission6.executeWithPermission(this, Manifest.permission.WRITE_EXTERNAL_STORAGE, this);
    }

    // ==========  deal with the result  ========== 
    @Override
    public void doAfterPermission() {
        Toast.makeText(this, "Permission granted", Toast.LENGTH_SHORT).show();
    }

    @Override
    public void userDenyPermission() {
        Toast.makeText(this, "Permission denied", Toast.LENGTH_SHORT).show();
        // normally, a dialog is better for user experience
        Permission6.startAppSettings(this, "ca.six.util.demo");
    }
}
```

#### How it work?
Actually, the `Permission6` is a Activity.

When you call `Permission.executeWithPermission()`, you are actually jump to the Permission6 class, and the Permission6 Activity will do the request task for you. 

When Permission6 gets the result, it will tell you throught the `IAfterDo` interface. 

The source is very short. You can see [the source here](https://github.com/songzhw/android-utils/tree/master/Permission6). 


#### Conclusion
Permission6 will make your request of permission very easy. In the meanwhile, you don't have to write too much boilerplate, and you don't declair your activity as an extension of some other specific Activity.




