# v2.0 : Compile-time ButterKnife(2)

In the previous post, I explained how to make your own ButterKnife. Now you know how to remove "findViewById()" in your codes.  

As a android develper, you may wrote the codes below so many times that it becomes a pain to you. 
```java
    btnOk.setOnClickListener(new OnClickListener{
            @Override
            public void onClick(View v) {
                // ... 
            }            
        }
    );
```

In this post, we'll build a more complex dependency injection framework. Now our own "ButterKnife" will start to deal with the "onClick()". 

And our goal is like this:
```java
@OnClick(R.id.hello) 
public void sayHello() {
    Toast.makeText(SimpleActivity.this, "Hello, views!", LENGTH_SHORT).show();
}
```

So, we need to generate a class like this:
```java
public class MainActivity$$Binds {
    public static void inject(MainActivity activity) {

        activity.tv = (android.widget.TextView) activity.findViewById(2131492971);
        activity.fab = (android.support.design.widget.FloatingActionButton) activity.findViewById(2131492970);

        activity.fab.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                activity.sayHello();
            }
        });

    }
}

```


Now let's start to work!

### Step 1. add another annotation in the "api" module
```java
@Retention(RetentionPolicy.CLASS)
@Target(ElementType.METHOD)
public @interface Click {
    String value();   
}

```


### Step 2. add the procedure in the "compile" module

add the code in the process() method:
```java
// 1. @Bind
for (Element element : roundEnv.getElementsAnnotatedWith(Bind.class)) {
    ...  ..  ..
}
// 2. @Click
for(Element element : roundEnv.getElementsAnnotatedWith(Click.class)) {
    if(element instanceof ExecutableElement){
        ExecutableElement method = (ExecutableElement) element;
        String clickVariName = method.getAnnotation(Click.class).value();
        String clickMethodName = method.getSimpleName().toString();
        MethodInfo methodInfo = new MethodInfo();
        methodInfo.variName = clickVariName;
        methodInfo.methodName = clickMethodName;
        methods.add(methodInfo);
    }
}

```


And then generate source codes:
```java
// 1. @Bind
for(FieldInfo variInfo : fields) {
    builder.addStatement("activity.$L = ($L) activity.findViewById($L)",
            variInfo.fieldName, variInfo.fieldType, variInfo.idRes);
}

// 2. @Click
for(MethodInfo methodInfo : methods){
    MethodSpec onClick = MethodSpec.methodBuilder("onClick")
            .addAnnotation(Override.class)
            .addModifiers(Modifier.PUBLIC)
            .addParameter(ClassName.get("android.view", "View"), "view")
            .returns(void.class)
            .addStatement("activity.$L()", methodInfo.methodName)
            .build();
    TypeSpec clickable = TypeSpec.anonymousClassBuilder("")
            .addSuperinterface(ClassName.get("android.view", "View.OnClickListener"))
            .addMethod(onClick)
            .build();
    builder.addStatement("activity.$L.setOnClickListener($L)", methodInfo.variName, clickable);
}

```

### Step 3. use the "Click" annotation in the "app" module
```java
@Click("fab")
public void showSnack() {
    Snackbar.make(fab, "butterknife tag 2.0", Snackbar.LENGTH_LONG)
            .setAction("Action", null).show();
}

```


Once you understand the last post, the previous codes are not difficult to you.  And after you add the "onClick" part, the "onTextChanged()", "onCheckChanged()" part is easy to add too. 









