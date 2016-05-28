# v1.0 : Compile-time ButterKnife

There are another different type annotation: compile-time annotation.  It means that errors in your annotation semantics can be detected early.  Most of all, the compile-time annotation can also improve performance.  Reading annotations requires the use of Java's reflection APIs, which can be a little expensive on Android. And Generated code can reduce startup time. 

## How to write an annotation processor
You need to use APT(Annotation Processing Tool) to write compile-time annotation. 

Hannes Dorfmann wrote a brilliant post, called "Annotation Processing 101". Please read it if you do not know how to make a compile-time annotation. [Here is the address](http://hannesdorfmann.com/annotation-processing/annotationprocessing101).   

In short, you have to 
* add apt, JavaPoet to your build.gradle
* write an annotation to get extra info
* write an processor extened from `AbstractProcessor`. You can read the annotation and generate source file in this processor.  Javac will do the real generating file job.
* use this annotatin in your Activity or other class.

## Introducing JavaPoet
Actually we will generated a java class to help you write the `findViewById()`. And [Square's JavaPoet](https://github.com/square/javapoet) is a tool that can generate java source file. 

Here is a simple example. If you want to generate a HelloWorld file:
```java
package com.example.helloworld;

public final class HelloWorld {
  public static void main(String[] args) {
    System.out.println("Hello, JavaPoet!");
  }
}
```
This is how you can generate it with JavaPoet:
```java
MethodSpec main = MethodSpec.methodBuilder("main")
    .addModifiers(Modifier.PUBLIC, Modifier.STATIC)
    .returns(void.class)
    .addParameter(String[].class, "args")
    .addStatement("$T.out.println($S)", System.class, "Hello, JavaPoet!")
    .build();

TypeSpec helloWorld = TypeSpec.classBuilder("HelloWorld")
    .addModifiers(Modifier.PUBLIC, Modifier.FINAL)
    .addMethod(main)
    .build();

JavaFile javaFile = JavaFile.builder("com.example.helloworld", helloWorld)
    .build();

javaFile.writeTo(System.out);
```
The codes are easy to read and use. More examples can be seen [in here](https://github.com/square/javapoet).

# v1.0 version

## Step 1. New a project.
Then I have the "app module" by default.

And I need to add the apt to our project.  So I add the following code in the build.gradle which is in the root directory of this project.
```groovy
buildScript {
    ...
    dependencies {
        ...
        classpath 'com.neenbedankt.gradle.plugins:android-apt:1.8'
    }
}
...
```



## Step 2. add a "api" module
Now I new a java library module, called "api".  I write only one class in this module. 
```java
@Retention(RetentionPolicy.CLASS)
@Target(ElementType.FIELD)
public @interface Bind {
    int value();
}
```

I will use it later in the Activity like this:
```java
@Bind(R.id.tv_main)
public TextView tv;
```

## Step 3. add a "compiler" module
This compiler moudle is still a java library module. 

#### Step 3-1. {project}/compiler/build.gradle
```groovy
apply plugin: 'java'
dependencies {
    compile fileTree(include: ['*.jar'], dir: 'libs')
    compile project(':api')
    compile 'com.google.auto.service:auto-service:1.0-rc2'
    compile 'com.squareup:javapoet:1.6.1'
}
```

`AutoService` : AutoService generates metadata for the developer, for any class annotated with @AutoService, avoiding typos, providing resistance to errors from refactoring, etc.

`JavaPoet` : Square's library for generate java source files more easier.

**Note**:  "compiler" module is middler layer tou our app. It only works in the javac, aka, the compile phrase.  So you can add heavy and useful library to this "compile" library as you want, such as guaua and apache library. The final APK will not exceed 65536 just because this "compile" module. 

#### Step 3-2. add a processor
```java
@AutoService(Processor.class)
public class BindProcessor extends AbstractProcessor {
    ... 
    @Override
    public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnv) {
        String fieldName = "", fieldType = "", activityLongName = "", activityName = "", packageName = "";
        int idRes = -1;
        TypeElement typeElement = null;
        for (Element element : roundEnv.getElementsAnnotatedWith(Bind.class)) {
            if (element instanceof VariableElement) {
                VariableElement vari = (VariableElement) element;
                idRes = vari.getAnnotation(Bind.class).value();
                fieldName = vari.getSimpleName().toString();
                fieldType = vari.asType().toString();
                log("szw variable : name = " + fieldName + " ; class = " + fieldType);   //=> "tv", "android.widget.TextView"
                typeElement = (TypeElement) vari.getEnclosingElement();
                PackageElement pkgElement = processingEnv.getElementUtils().getPackageOf(typeElement);
                activityLongName = typeElement.getQualifiedName().toString();
                activityName = typeElement.getSimpleName().toString();
                packageName = pkgElement.getQualifiedName().toString();
                log("szw qualifiedName = " + activityLongName + " ; " + activityName + " ; " + packageName); //=> "ca.six.demo.MainActivity", "MainActivity","ca.six.demo"
            }
        }
        if(packageName == null || packageName.equals("")){
            return true;
        }


        // start to generate a java source file
        try {           
            MethodSpec.Builder builder = MethodSpec.methodBuilder("inject")
                    .addModifiers(Modifier.PUBLIC, Modifier.STATIC)
                    .returns(void.class);
            builder.addParameter(ClassName.get(packageName, activityName), "activity") ;
            builder.addStatement("activity.$L = ($L) activity.findViewById($L)", fieldName, fieldType, idRes);
            MethodSpec sInject = builder.build();
            TypeSpec innerClz = TypeSpec.classBuilder(activityName+"$$Binds")
                    .addModifiers(Modifier.PUBLIC)
                    .addMethod(sInject)
                    .build();
            JavaFile file = JavaFile.builder(packageName, innerClz)
                    .build();
            log("Log to writeTo() : " + processingEnv.getFiler().toString());
            file.writeTo(processingEnv.getFiler());
            // generate another one in the Desktop for sure
            file.writeTo(new File(System.getProperty("user.home") + "/Desktop/"));
        } catch (Exception e) {
            error("Error to generateCode() : " + processingEnv.getFiler().toString());
            e.printStackTrace();
        }
        return false;
    }
    private void log(String str) {
        messager.printMessage(Diagnostic.Kind.NOTE, str);
    }
    private void error(String str) {
        messager.printMessage(Diagnostic.Kind.ERROR, str);
    }
}
```

Quite a long list, right? Don't worry, I will introduce the important detail later.



## Step 4. 