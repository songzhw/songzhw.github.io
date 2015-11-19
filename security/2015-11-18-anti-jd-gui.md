## How to make jd-gui failed?

Android security is a big problem for us. Nowaday, jd-gui, apktool, IDA pro can easily decompile our app, and therefore our most important source are leaked.

How can we stop these? Oh, this is a really big question. Today, we focus on the java source: How to fail the jd-gui?

## Thought
We found the jd-gui has a problem of decompiling some function.<br/>
![](/imgs/20151118_01.jpg)<br/>
So Why not extract these codes and add them to our app, then the jd-gui will have problems to decompile the project.

## Steps
##### 1. add a TTT class
```java
import android.content.Context;

public class TTT {
    public static void set(Context ctx, String key, String value) {
    }
}
```

##### 2. add these codes to the function that you want to hide
```java
        /*****************************************************************/
        // to anti jd-gui
        switch(0){
            case 1001:
                JSONObject obj;
                String date = null;
                boolean isClose = false;
                try {
                    obj = new JSONObject("");
                    date = obj.getString("date");
                    isClose = obj.getBoolean("isClose");
                } catch (JSONException e) {
                    e.printStackTrace();
                }
                TTT.set(null, "", date);
                break;
        }
        /*****************************************************************/
```

##### 3. Open the proguard. And your release apk will have the anti-jd-gui power.
The specific move is to change the value of "minifyEnabled" from "false" to "true".

```groovy
buildTypes{
    release {
        minifyEnabled true
        proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
    }
}
```

## result
```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        /*****************************************************************/
        // to anti jd-gui
        switch(0){
            case 1001:
                JSONObject obj;
                String date = null;
                boolean isClose = false;
                try {
                    obj = new JSONObject("");
                    date = obj.getString("date");
                    isClose = obj.getBoolean("isClose");
                } catch (JSONException e) {
                    e.printStackTrace();
                }
                TTT.set(null, "", date);
                break;
        }
        /*****************************************************************/
    }
}

```

Now my MainActivity#onCreate() was hidden.
![](/imgs/20151118_02.jpg)


## Question
How about if we want to hide all the function?<p>
: Right now, I still have no direct answer.<br/>
 All I can do is to to recommand you to use this trick in these high-sensitive function, like the key-generating function, the encryption function and so on.


## Reference
http://www.cnblogs.com/ijiami/p/3407989.html

## Another similar blog
http://www.willhackforsushi.com/?p=562