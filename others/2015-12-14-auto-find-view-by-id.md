# 01. Tool to generate code boilerplate

In this blog, I will introduce how to generate findViewByID() and setOnClickListener() code automatically.

## Boilerplat

A normal Activity might like this:
```java
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_two_button);


        btnClick = (Button)findViewById(R.id.btnClick);
        btnClick.setOnClickListener(new OnClickListener(){
            public void onClick(View v){
                ...
            }
        }


        btnTarget = (Button)findViewById(R.id.btnTarget);
        btnTarget.setOnClickListener(new OnClickListener(){
            public void onClick(View v){
                ...
            }
        }
    }

```

A complex layout might have dozens of these bored lines. I really hates this. As a software engineer, I believe we definitely have a better way.

## A script to resolve this problem

### introduction
To solve this problem , I write a Groovy script to automatically generate Activity class for us.

* **Input**

    a layout xml file

* **Output**

    a Activity class (java class)
<p>

If you want the script to be more smart, you can name the id of View in xml file applying to my rules:

*  if the id of view ends with ```"_x"```, it means it will not be listed as a member in Class
* if the id of view ends with ```"_c"```, it means it will be listed as a member in Class, and it will be add the OnClickListener
* ``` "android:id" ``` should be the first attribute of a View in the layout xml file


### usage
1. paste your layout xml content to "layout.xml"
2. run the "auto_findview.bat",  or "auto_findview.bat"

### result
```java
private Button btnSendSms;
private TextView tvBankError;
private EditText etInputPwd;

etInputPwd = (EditText) findViewById(R.id.et_pay_input_pwd);
tvBankError = (TextView) findViewById(R.id.tv_pay_bank_error);
btnSendSms = (Button) findViewById(R.id.btn_pay_send_sms_c);
btnSendSms.setOnClickListener(this);

@Override
public void onClick(View v) {
	switch (v.getId()) {
		case R.id.btn_pay_send_sms_c:

			break;
		default:
			break;
	}
}

```


### Link:

https://github.com/songzhw/android-toolkit/tree/master/auto-findView


# 02.  Another option : kotlin 

Kotlin extension for Android omit the findViewById for you , which is really convenient. 

I stronly recommemd you to try it!