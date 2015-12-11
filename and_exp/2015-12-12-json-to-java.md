# Tool to parse JSON

JSON is a lightweight data and is used widely in the development. 

## a normal sample dealing with json

```json
[
    {
        "name": "name0", 
        "age": 0
    }, 
    {
        "name": "name1", 
        "age": 5
    }, 
    {
        "name": "name2", 
        "age": 10
    }
]

```
When we have a json string like thre previous json, we can parese it manually, or use Gson/FastJson/Jackson library to parse it semi-automatically. 

Why I use the word "semi-automatically"?<br/>
because when you want to use GSON to parse a json to java object, you have to write a java file like this:
```java
public class Person {

    public String name;
    public int age;
    // or private members with getter and setter

}
```

And then in the Activity, or response class, to pass the json to the GSON, and then get a java POJO instance.
```java
Person person = gson.fromJson(str, Person.class);
```

Yes, if I want to use GSON, i have to write a java POJO class first.

If I don't use GSON, I then have to parse the json string all by myself, which is more complex.
```java
JSONObject str = new JSONObject(jsonStr);
String name = str.optString("name");
int age = str.optInt("age");
```

## How about write a script?
Since the previouse two ways are not good enough, how about I write a script to do the job?

I first need a input, which is a json string, or a file whose content is a json string<br/>
Then the expected output is some files of Java POJO classes, like "Person.java"

Here is my script, which is shared in GitHub:<br/>
https://github.com/songzhw/android-toolkit/tree/master/json-resp

## Conclusion
the advantage of this script:
1. I import no library, which means I do not import thousands of method count. As we know, method count limit (65536) is a proglem for us, and we should be more careful with the method count.
2. I do not need write a java POJO class first. 
3. It's easy to change the script to adjust some other projects. And the alternative ways, like GSON or Intellij IDEA plugin is not that convinent.




