# Kotlin help to improve our project structure

I recently has learned some knowledge about fingerprint in Android 6.x. And I have to say, fingerprint code in Java is ugly. And How can I improve it?

## Fingerprint -- Java version

Mariotaku has greatly simplize the fingerprint code, whereas the google sampe is really big and not so readable(it uses dragger, which is not familiar with everyone).

However, when I try to apply fingerprint in more than one Activity, I realized I have to write a lot of reduplicate code

I need to check "use_fingerprint" permission. If I have not such permission, I have to request it.<br/>

the *checkSelfPermission()* , *requestPermissions()*, *onRequestPermissionsResult()* all are the functions of *Activity* class, which means I can not abstract them out for reusing.<br/>

So if I want to reuse the three functions, and avoid reduplication, I have no choice but to write a parent Activity. However, I think **"Composition Over Inheritance"** is a pattern that we should try. And this is the opposite.

```java

/**
 * Created by songzhw on 2015/12/25
 * Note: You can rewrite this code, like "extends YourBaseActivity"
 *
 *
 *  public methods :
 *     * onAuthenSucc()
 *     * isFingerprintAvailable()
 *
 */
public class FingerpringBaseActivity extends Activity {
    private static final int REQUEST_PERMISSION = 110;

    private FingerpringBaseActivity myself;
    private FingerprintManager fingerprintMgr;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        myself = this;
        fingerprintMgr = getSystemService(FingerprintManager.class); // getSystemService() : API 23

    }

    // ====================================================
    // 1. detect fingerprint

    protected boolean isFingerprintAvailable(){
        boolean hasPersmission = checkSelfPermission(Manifest.permission.USE_FINGERPRINT) == PackageManager.PERMISSION_GRANTED;
        boolean hasHardware = fingerprintMgr.isHardwareDetected();
        boolean hasFingerprints = fingerprintMgr.hasEnrolledFingerprints();
        if(hasPersmission && hasHardware && hasFingerprints){
            return true;
        } else {
            if(! hasPersmission){
                final String[] permissions = {Manifest.permission.USE_FINGERPRINT};
                requestPermissions(permissions, REQUEST_PERMISSION);
            } else if(!hasHardware){ // hasPermission && !hasHardware
                Toast.makeText(this, "This device doesn't support Fingerprint authentication", Toast.LENGTH_LONG).show();
            } else if(!hasFingerprints) { // hasPermssion && hasHardware && !hasFingerprints
                Toast.makeText(this, "You haven't enrolled any fingerprint, go to System Settings -> Security -> Fingerprint", Toast.LENGTH_LONG).show();
            }
            return false;
        }
    }

    @Override
    public void onRequestPermissionsResult(final int requestCode, final String[] permissions, final int[] grantResults) {
        switch (requestCode) {
            case REQUEST_PERMISSION: {
                if (grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                    isFingerprintAvailable();
                } else {
                    Toast.makeText(this, "Please give app fingerprint permission", Toast.LENGTH_LONG).show();
                }
                break;
            }
        }
    }




    // ====================================================
    // 2. Fingerprint Result

    public void onAuthenError(CharSequence msg){

    }

    public void onAuthenFail(){

    }

    public void onAuthenHelp(CharSequence msg){

    }

    public void onAuthenSucc(FingerprintManager.AuthenticationResult result){

    }

    public void authenFingerprint(){
        try {
            fingerprintMgr.authenticate(null, null, 0, fingerResult, null);
        } catch(SecurityException e){
            Toast.makeText(this, "fingerpirnt error : Permmision, hardward, or has no fingerprint", Toast.LENGTH_LONG).show();
        }
    }


    protected  FingerprintManager.AuthenticationCallback fingerResult = new FingerprintManager.AuthenticationCallback() {
        @Override
        public void onAuthenticationError(int errorCode, CharSequence errString) {
            myself.onAuthenError(errString);
        }

        @Override
        public void onAuthenticationHelp(int helpCode, CharSequence helpString) {
            myself.onAuthenHelp(helpString);
        }

        @Override
        public void onAuthenticationSucceeded(FingerprintManager.AuthenticationResult result) {
            myself.onAuthenSucc(result);
        }

        @Override
        public void onAuthenticationFailed() {
            myself.onAuthenFail();
        }
    };


}


```

**Note** : Since java can not pass function as a first-class citizen, I do not transmit the error handling message out and just toast them.<br/>
This is not right. What if I have another handling, rather than toasting?

I will talk about how to fix it in Kotlin later.


Then I want to use fingerprint, I only have to write a Activity whose parent is just the FingerprintBaseActivity.

```java
/**
 * Created by songzhw on 2015/12/25
 */
public class MyFingerprintTestActivity extends FingerpringBaseActivity {
    private ImageView iv;
    private TextView tv;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_my_fingerprint);

        iv = (ImageView) findViewById(R.id.ivMyFingerprint);
        tv = (TextView) findViewById(R.id.tvMyFingerprint);

    }

    public void clickAuthen(View v){
        boolean isFingerprintEnable = isFingerprintAvailable();
        if(isFingerprintEnable){
            tv.setText("Start to authenticate your fingerprint ...");
            iv.setImageResource(R.drawable.ic_fingerprint);
            authenFingerprint();
        }
    }

    // Touch the fingerprint zone will get no response
    // msg is like "尝试次数过多， 请稍后重试 "
    @Override
    public void onAuthenError(CharSequence msg) {
        tv.setText("[Error]" + msg + " ;  Please try again");
        iv.setImageResource(R.drawable.ic_fingerprint_error);
    }


    // After getting this, you can retry and touch the fingerprint zone again.
    @Override
    public void onAuthenFail() {
        tv.setText("[Fail]" );
        iv.setImageResource(R.drawable.ic_fingerprint_error);
    }

    @Override
    public void onAuthenHelp(CharSequence msg) {
        tv.setText("[Help]" + msg);
    }

    @Override
    public void onAuthenSucc(FingerprintManager.AuthenticationResult result) {
        FingerprintManager.CryptoObject crypto = result.getCryptoObject();
        tv.setText("[Succ]" + crypto);
        iv.setImageResource(R.drawable.ic_fingerprint_success);
    }
}
```


## Fingerprint -- Kotlin version

The disadvantage of the preceding java code :

1. violate the **"Composition Over Inheritance"**  principle. <br/>
Every Activity that want to use fingerprint is asked to extend from FingerBaseActivity.

2. Java cannot pass function as a first-class citizen. And an interface with several methods is too complicated to write.

Since kotlin can extend system classes, and the function in kotlin is first-class citizen. Can I improve the structure as a better, cleaner project?

Yes, I can try.  My thought is like this:

```kotlin
fun Activity.fingerprint(){
    if(hasNoPermission){
        this.requestPermissionsManifest.permission.USE_FINGERPRINT, 110)

        // override the onRequestPermissionResult() function
        this.onRequestPermissionsResult = {requestCode: Int, permissions: Array<out String>, grantResults: IntArray ->
            // ....
        }
    }
    else if(...) {...}
    else {
        fingerMgr.authen(null, null, 0, callback, null);
    }
}

```

However, I found the ```this.onRequestPermissionsResult = {...}``` is reported to be error by the kotlin compiler.

And I found out I cannot override the exiting method in a system class ( https://kotlinlang.org/docs/reference/extensions.html ):

```kotlin
class C {
    fun foo() { println("member") }
}

fun C.foo() { println("extension") }

c.foo() //=> member
```

The kotlin group said "If there's a member and extension of the same type both applicable to given arguments, **a member always win!**"

So now I understand , I cannot override the member function of a system class in the kotlin world. (p.s : Ruby is okay with it. Ruby is of course my favorite programming language.)

Even we can not override the memeber function, I still can make the project better.


**Step 1**: Add a util class : cn/six/payx/util/FingerprintUtil.kt

```kotlin
package cn.six.payx.util

import android.Manifest
import android.app.Activity
import android.content.pm.PackageManager
import android.hardware.fingerprint.FingerprintManager

private val REQUEST_FINGERPRINT = 120

/*
Note that, since extensions do not actually insert members into classes,
there’s no efficient way for an extension property to have a backing field.
This is why initializers are not allowed for extension properties.
Their behavior can only be defined by explicitly providing getters/setters.
*/
val Activity.fingerMgr : FingerprintManager
    get() = getSystemService(FingerprintManager::class.java)


fun Activity.isFingerprintAvailable() : Boolean {
    val hasPersmission = this.checkSelfPermission(Manifest.permission.USE_FINGERPRINT) == PackageManager.PERMISSION_GRANTED;
    val hasHardware = fingerMgr.isHardwareDetected();
    val hasFingerprints = fingerMgr.hasEnrolledFingerprints();
    if(hasPersmission && hasHardware && hasFingerprints){
        return true
    } else {
        if(! hasPersmission){
            val permissions = arrayOf(Manifest.permission.USE_FINGERPRINT)
            this.requestPermissions(permissions, REQUEST_FINGERPRINT);

            // [!!!] If there’s a member and extension of the same type both applicable to given arguments, a member always wins!
//            this.onRequestPermissionsResult : (...)-> Unit = {
//            }



        } else if(!hasHardware){ // hasPermission && !hasHardware
            showToast("This device doesn't support Fingerprint authentication")
        } else if(!hasFingerprints) { // hasPermssion && hasHardware && !hasFingerprints
            showToast("You haven't enrolled any fingerprint, go to System Settings -> Security -> Fingerprint")
        }
    }
    return false
}

fun Activity.onPermitted(requestCode: Int, permissions: Array<out String>, grantResults: IntArray) {
    when(requestCode){
        REQUEST_FINGERPRINT ->{
            if (grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                isFingerprintAvailable()
            } else {
                showToast("Please give app fingerprint permission")
            }
        }
    }
}

fun FingerprintManager.authen(onSucc : (FingerprintManager.AuthenticationResult?)->Unit,
                              onHelp : (CharSequence?)->Unit,
                              onFail : ()->Unit,
                              onError : (CharSequence?)->Unit){

    this.authenticate(null, null, 0,
            object : FingerprintManager.AuthenticationCallback(){
                override fun onAuthenticationSucceeded(result: FingerprintManager.AuthenticationResult?) {
                    onSucc(result)
                }

                override fun onAuthenticationHelp(helpCode: Int, helpString: CharSequence?) {
                    onHelp(helpString)
                }

                // After getting this, you can retry and touch the fingerprint zone again.
                override fun onAuthenticationFailed() {
                    onFail()
                }

                // "尝试次数过多， 请稍后重试 "
                override fun onAuthenticationError(errorCode: Int, errString: CharSequence?) {
                    onError(errString)
                }
            }
            , null)
}

```


**Step 2**: write a Activity to use fingerprint

```kotlin
import cn.six.payx.util.authen
import cn.six.payx.util.fingerMgr
import cn.six.payx.util.onPermitted
import kotlinx.android.synthetic.activity_balance.*

public class BalanceActivity : BaseActivity(){

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_balance)

        tvBalanceInfo.setText("Your balance is encrypted. \nTouch the fingerprint zone to see it.....")
        tvBalanceInfo.setTextColor(Color.RED)

        fingerMgr.authen(
                {
                    tvBalance.setText("$88.88")
                    tvBalanceInfo.setText("")
                },
                {tvBalanceInfo.setText(it)},
                {tvBalanceInfo.setText("failed. Please try again!")},
                {tvBalanceInfo.setText("ERROR! : $it")}
        )
    }

    override fun onRequestPermissionsResult(requestCode: Int, permissions: Array<out String>, grantResults: IntArray) {
        onPermitted(requestCode, permissions, grantResults)
    }
}
```

After the two steps, we apply **"Composition Over Inheritance"**  principle to our project.
And since we can transfer function, the readability of our code is better.


## Conclusion
A modern programming language with lambda, first-class citizen function, monkey patch will make our project structure better. This kotlin code is an example. Actually, a rough example.

I have not deal with the Android belows API Level 23, or the error handling in thie post. And I will continue to make it better later.


**2015.12.29** 
I already add the API Level solution and the error handling codes in https://github.com/songzhw/CleanBeta-Kotlin/tree/master/app/src/main/kotlin/cn/six/payx

The codes includes :
  *  BalanceActivity
  *  IBalancePresenter
  *  BalanceLPresenter
  *  BalanceMPresenter

