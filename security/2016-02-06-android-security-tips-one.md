Android faces a lot of security hole nowaday. And as a android app developer, we should know some secruity features and knows how to fix these security holes.

I will write several posts about the security tips in the android app developing. This is the starter.


## 1. AndroidManifest

Activity/Service/Broadcast has a attribute called "android:exported" in the AndroidManifest.xml. This attribute is tricky. The default value of this attribute is false. This means other app cannot access your Activity/Service/Broadcast, which is good for your security. 

However, if you add a ```<intent-filter>``` to your Activity/Service/Broadcast, this means your Activity/Service/Broadcast's "andorid:exported" attribute is assigned a value of "true". This is a tricky one, and you have to remember it.



## 2. WebView

### 2.1 JS Bridge
WebView has many problem, especially the low version one. In the android that is lower than 4.2, ```webview.adJavascriptInterface(javaObject)``` is a well-known security hole. 

When you add this ```javaObject``` to the webview,  the webview is actually get the  ```javaObject.getClass()``` , therefore it can get Linux permission to do something bad, like to show the whole content of the SD card. 

[Solution]
1. For the Android which is >= 4.2, you can use the ```@javascriptInterface``` annotation to make sure this function can be accessed by Java. Like this

```java
class JsBridge{
	@JavascriptInterface
	public String getLocation(){
		// ... get the locatio and give it to javascript
	}
}
```

2. For the Android which is < 4.2
WebChromeClient has three functions :
  * onJsPromot()
  * onJsAlert()
  * onJsConfirm()

You can use one of them to comunicate JavaScript from Java. Like this:

```java
public boolean onJsAlert(WebView webview, String url, String message, String defaultValue, JsAlertResult result) {
	if( isMyProtocol(message)){
		// do something, like getting the location for js
		return true; // "return true" means there are no alert dialog again in javascript
	}
	return super.onJsAlert(webview, url, message, defaultValue, result);
}
```

More details : http://blog.csdn.net/leehong2005/article/details/11808557

### 2.2 WebView's cache

When you fill the form (like the login form) and choose to save the data, the form data is actually saved in the data/data/<app package>/databases/webview.db。 

If your phone is not a root phone, that's okay。 But if your phone is rooted, then other app may have the chance to hack you.

So if you want to fix this, you can do this.

```java
webview.getSettings().setPassword(false);
```

More details : http://www.wooyun.org/bugs/wooyun-2013-020246

## 3. Https


## 4. Storage


## 5. AIDL


## 