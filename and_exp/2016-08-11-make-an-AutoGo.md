[AutoGo](https://github.com/TellH/AutoGo) is a libray to help you jump to another Activity elegantly. Today, let's meet it and Please let me tell you how to make your AutoGo.

### Introduction to AutoGo
Here is how you use it.
- The Activity you will jump to 

```java
public class TargetActivity extends AppCompatActivity {
    //the annotated field should not be private
    @IntentValue("myName") String name;
    @IntentValue int age;
    @IntentValue ArrayList<String> friends;
    ......
}
```

- The Activity you will jump from

```java
        AutoGo.from(SrcActivity.this)
                .gotoTargetActivity()
                .age(18)
                .myName("tlh")
                .friends(friends)
                .go();
```

Isn't it pretty?

### How to make our own AutoGo
Actually, the previous three posts named `"How to make your own ButterKnife */3` already tells you how to make a tool using Java's annotatin at compile time. And AutoGo is also such a tool writen by annotation at compile time. 

#### 1. Define an Annotation

```java
@Retention(RetentionPolicy.CLASS) 
@Target(ElementType.FIELD)
public @interface IntentValue {
    String value() default "";
}
```

#### 2. write your own processor
Remember, you have to write an extension of `AbstractProcessor`, and do whatever you want in the `process(annotations, roundEnv)` method.

Here is the two steps you need to follow:
##### 2.1 analyze the annotation and its context
By "context", I mean the class who has a field defined by the annotation, the package, the name of the field, the type of the field, etc. 

You can get all these information from `roundEnv.getElementsAnnotatedWith(Auto.class))` method. And you should save all the information you need in a ArrayList.

##### 2.2 generate the java file
Now you have the information you need from the Annotation, now you can generate the java class file. 

```java
        AutoGo.from(SrcActivity.this)
                .gotoTargetActivity()
                .age(18)
                .myName("tlh")
                .friends(friends)
                .go();
```

If you want to generage the AutoGo class, you apparently need to generate `age()`, `myName()`,`firends()` methods.  Is this hard? No, you already know what field has the annotation, yes, the `age`,`MyName`,`friedns` field. 

Now what you should do is just to new three `MethodSpec` (a class of [JavaPoet](https://github.com/square/javapoet)), and add them to `AutoGo` class.

Now we know how to make your own AutoGo. I recommend you to try it out by yourself. The knowledge you learn from book cannot compare to the knowledge you learn from practice. Besides, throngh this practice, you will also learn a lot about annotation at compile time. And if you think more about the annotation, you may invent you own tool. 