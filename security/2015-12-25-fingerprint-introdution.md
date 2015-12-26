# Android Fingerprint
Android 6.0 has imported the fingerprint. As a person who is firstly meet the fingerprint in Android, I want to introduce fingerprint from the very begining.

##  1. Set the fingerprint
When I got the Nexus 6P, I boot the phone.<p>

step 1 : You tell Android6.0 that you want to have some kind of password to unlokc your phone. The choices are fingerprint, pin code, lock pattern. <br/>
        -- I choosed "FingerPrint".

step 2 : Android6.0 asked you to set pin code / lock pattern firstly.<br/>
        -- I choosed "lock pattern" and set the lock pattern.

setp 3 : Before letting you set the fingerprint, Android specially tell you "Fingerprint is not as secure as pin code or lock pattern".<br/>
![](/imgs/20151225_01.jpg)

(translation of the text coverd by the red frame: "Fingerprint is not as secure as locak pattern or pin code.")


step 4 : gathering your one fingerprint serveral times, to make sure the result is accurate.<br/>

p.s : you can add more fingerprints later in the "settings --> security --> Nexus Imprints"


## 2. A simplest example of fingerprint

```java
/**
 * Created by songzhw on 2015/12/24
 */
public class BoolFingerActivity extends Activity {
    private Button btnPress;
    private TextView tvLog;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.actv_mariotaku_finger);
        fingerprintMgr = getSystemService(FingerprintManager.class);

        btnPress.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                fingerprintMgr.authenticate(null, null, 0, authenticator, null);
                tvLog.setText("Press your finger !");
            }
        });
    }

    private FingerprintManager.AuthenticationCallback authenticator = new FingerprintManager.AuthenticationCallback() {
        @Override
        public void onAuthenticationSucceeded(FingerprintManager.AuthenticationResult result) {
            tvLog.setText("FINGERPRINT --> onAuthenticationSucceeded()\n\n");
        }

        @Override
        public void onAuthenticationError(int errorCode, CharSequence errString) {
            tvLog.setText("FINGERPRINT --> onAuthenticationError() : " + errString + "\n\n");
        }

        @Override
        public void onAuthenticationHelp(int helpCode, CharSequence helpString) {
            tvLog.setText("FINGERPRINT --> onAuthenticationHelp() : " + helpString + "\n\n");
        }

        @Override
        public void onAuthenticationFailed() {
            tvLog.setText("FINGERPRINT --> onAuthenticationFailed()\n\n");
        }
    };
}

```

That's simple, right?
You only have to call "fingerMgr.authenticate(..., callback, ...)", and the callback will get the result later.

## 3. More complicated example
In the real world, you have to make sure

(1). this phone has the fingerprint function
```java
	boolean = fingerPrintMgr.isHardwareDetected();
```

(2). Or phone users already enroll at least one fingerprint

```java
	boolean = fingerPrintMgr.hasEnrollFingerprints();
```

(3). Or do you get the "USE_FINGERPRINT" permission. And if you don't, you have to request the permission

```java
	if (checkSelfPermission(Manifest.permission.USE_FINGERPRINT) != PackageManager.PERMISSION_GRANTED) {
        final String[] permissions = {Manifest.permission.USE_FINGERPRINT};
        requestPermissions(permissions, REQUEST_PERMISSION);  // REQUEST_PERMISSION is a final variable whose value is 120
	}


    @Override
    public void onRequestPermissionsResult(final int requestCode, final String[] permissions, final int[] grantResults) {
        switch (requestCode) {
            case REQUEST_PERMISSION: {
                if (grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                    // ok, you are granted
                } else {
                    // error
                }
                break;
            }
        }
    }
```

<p>
To see the source of this example, please see @mariotaku's repo:

https://github.com/mariotaku/FingerprintSample/blob/master/src/main/java/org/mariotaku/fingprint/sample/MainActivity.java


## reference
https://github.com/mariotaku/FingerprintSample
A simplized example, compared to the googleSample/androidFingerprint-dialog