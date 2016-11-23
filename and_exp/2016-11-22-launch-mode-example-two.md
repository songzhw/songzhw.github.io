I've got a bug about the Activity stack recently, and I fixed it using the knowledge of launch mode and Intent's flag. This post will introduce how I locate the reason, and how I fixed it. 

## What's the problem I got?
We have an Activity (let's call it A) which could contains multip Fragment. When you click the button on Fragment (let's call it B), what it does is jump to A, so A could launch another Fragment, "C" it is.  Oh, forget to mention, this "jump" action actually has an "Clear_top" flag in its Intent. 

Let's see the code.

This is Activity A. It will add a Fragment according to the fragmentId it gets.  And it will set the isEnable field to false on its onDestroy() method

```java
// [PaymentActivity]
    public static boolean isEnable = true; //you can think of it as a flag, we may need it somewhere in real project

    public void showFragment(int fragmentId) {
        FragmentManager fragmentManager =getFragmentManager();
        FragmentTransaction transaction = fragmentManager.beginTransaction();

        if(fragmentId == ACTION_PAY){
            PayFragment frag = new PayFragment();
            transaction.add(R.id.flayPay, frag);
        } else if(fragmentId == ACTION_RESULT){
            ResultFragment frag = new ResultFragment();
            transaction.add(R.id.flayPay, frag);
        } else {
            return;
        }

        transaction.commit();
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        isEnable = false;
    }


```

This is Fragment B.

```java
// [PayFragment]
    @Override
    public void onClick(View v) {
        if(v.getId() == R.id.btn_simple) {
            actv.showFragment(PaymentActivity.ACTION_RESULT);
        } else {
            tv.setText("PayFragment : isEanble = "+actv.isEnable);
        }
    }

```

This is Fragment C. 

```java
// [ResultFragment]
    @Override
    public void onClick(View v) {
        Intent it = new Intent(actv, PaymentActivity.class);
        it.addFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP);
        actv.startActivity(it);
    }

```



Now, when we jumps to C from B, we found the ..... .

Why?

## Process to locate the reason

### Intent.FLAG_ACTIVITY_CLEAR_TOP
I used to believed, if you set "Intent.FLAG_ACTIVITY_CLEAR_TOP", that means it will kill all the activity on top of your target activity, and bring your target Activity to the front/top. 

Yes, this is right. But what do you mean by "bring your target Activity to the front/top"? If it does so, which lifecycle methods will be called? onCreate()? onReseum()? onStart()? or onNewIntent()?

I never thought of it before, now I have a change to get to know it. So I did some small experience about it. 

#### experience 01 : clear_top


#### experience 02 : celar_top


#### conclusion of clear_top


## Code
Code is in here : (github.com/songzhw)[https://github.com/songzhw/AndroidAdvanced/tree/master/Advanced/app/src/main/java/cn/six/adv/launch_mode/clear_top_bug].

