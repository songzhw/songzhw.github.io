# How to build a clean, test project for your app?

I introduce some test tools for you to start applying test to your app.  I do not want to talk too much the detail of using Espresso or UI Automator, since the introductions are everywhere in the Internet. Instead, I want to introduce a nice architecture of one test project.


## 1. The normal development and test codes
### 1.1 app/main

Firstly, I try to build a common request and its listener for responses.

```java
[IRespListener]
public interface IRespListener {
    void onResponsed(String resp);
}
```

Then I write a new class called **BaseRequest** that is a subclass of AsyncTask. Therefore, I do not have to write a thread pool to handle the request queue, and can easily transfer from the main thread to the work thread. 

```java
[BaseRequest]
public class BaseRequest extends AsyncTask<String, Void, String> {
    private String url;
    private IRespListener listener;

    public void startRequest(String url, IRespListener alistener) {
        this.url = url;
        this.listener = alistener;
        this.execute();
    }

    @Override
    protected String doInBackground(String... params) {
        HttpEngine httpEngine = new HttpEngine();
        return httpEngine.sendHttpGetRequest(url);
    }

    @Override
    protected void onPostExecute(String result) {
        if (listener != null) {
            listener.onResponsed(result);
        }
    }

}
```

The HttpEngine class is simple, just use the ```HttpUrlConnection``` to access our url:

```java
    public String sendHttpGetRequest(String url) {
        HttpURLConnection http = initHttpConnection(url);

        http.setRequestMethod("GET");
        int responseCode = http.getResponseCode();
        BufferedReader is = null;
        if (responseCode == HttpURLConnection.HTTP_OK) {
            is = new BufferedReader(new InputStreamReader(http.getInputStream()));
        } else {
            is = new BufferedReader(new InputStreamReader(http.getErrorStream()));
        }

        StringBuilder s = new StringBuilder();
        char[] buf = new char[1024];
        int num = -1;
        while ((num = is.read(buf)) != -1) {
            s.append(buf, 0, num);
        }
        String str = s.toString();

        is.close();
        http.disconnect();
        return str;
    }
```

The above three classes are our simple Http framework. It's not as powerful as OkHttp, but it is simple and easy to use. 

```java
public class GUserActivity extends AppCompatActivity implements IRespListener {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        FloatingActionButton fab = (FloatingActionButton) findViewById(R.id.fab);
        fab.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                startRequest();
            }
        });
    }


    private void startRequest() {
    	BaseRequest req = new BaseRequest();
        String url = "https://api.github.com/users/songzhw";
        req.startRequest(url, this);
    }


    @Override
    public void onResponsed(String resp) {
        final User user = new Gson().fromJson(resp, User.class);
        tv.setText(user.name);
    }
}
```

It's easy, right?

###  1.2. problems comes
Our Espresso is easy to write too. 

However, in some cases, I do not want to use the real server data to test. I prefer the mock data. Reasons are listed as follow:
1. real server data is slow and dependecy to the stability of the network
2. real server data is not easy to modify. You have to modify the server database or something. 
3. some real server api is different. Take the paying api for example, it takes an orderId as a argument, then pay the order. However, the orderId is a one-time thing. When you finished paying this bill, the orderId is no longer legal, which means you have to apply a new orderId every time you run a test case. Obviously, this is a disaster for us. 

So, the mock data is easy to use, and to modify, and is stable. So we do need to use the mock data in the test. 


### 1.3 refactor the whole project
To meet this requirement, I have to modify the app/main directory a little bit:

First of all, I have to create a class to hold all the mock json data in one place. So our mock data is managed by only one holder, which is, of course, a life saver in the future. 

```java
// app/src/main/.../MockApiRepo.java
public class MockApiRepo {
    public static final String API_USER = "{\n" +
            "name: \"testInRetrofit\",\n" +
            "location: \"Toronto, Canada\",\n" +
            "public_repos: 8\n" +
            "}";
}
```

Secondly, I add the debug branch to the Activity. 

```java
[Activity]
    private void startRequest() {
    	if(isDebugForTest){
    		onResponsed(MockApiRepo.API_USER);
    	} else{
	    	BaseRequest req = new BaseRequest();
	        String url = "https://api.github.com/users/songzhw";
	        req.startRequest(url, this);
        }
    }

    @Override
    public void onResponsed(String resp) {
        final User user = new Gson().fromJson(resp, User.class);
        tv.setText(user.name);
    } 
```

Finally, I refactor the test:

```java
//app/src/androidTest/.../UserTest.java


```

## 2. Start to build a cleaner test project

```java

```