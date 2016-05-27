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


