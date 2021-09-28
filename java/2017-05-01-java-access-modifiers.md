## Background

Today I want to create a subclass of "AppCompatDelegateImplV14". But I found out the definition of this class is :
```java
package android.support.v7.app;

class AppCompatDelegateImplV14 extends AppCompatDelegateImplV14 { ...}
```

And I failed to try to extend it:
```java
package ca.six.someapp;

public class MyAppComplatDelegateImpl extends AppCompatDelegateImplV14 {...}
```

I then realized that's because the access modifier of `AppCompatDelegateImpl` is default. 
The `AppCompatDelegateImpl` class is in the "android.support.v7.app" package, and my `MyAppComplatDelegateImpl` is in my "ca.six.demo" package.  Since there are two different package, so it's illegal to extend the default class. 

## Java Access Modifier
There is a famous table about the java access modifiers:

![](./_image/2017-04-30-22-45-37.jpg)

But I have to say, this table is not completed. The biggest question about this table is what is "accessible".  

### 1. For Class 
The access modifier can only be public or default.
---->> For the class with default access modifier, you can not extend it outside its package.

### 2. For Method Invoke
Protected == Default : you can invoke the protected/default method only if you are in the same pakcage as the method you wanted to invoke. 

### 3. For Method Override
For the subclass in different package with the parent class, subclass can overide the protected methods, but not the default methods.

### 4. For Field
In the same package: you can use the projected/default field.
In different packages: Subclass can only use protected field by "this.field". Subclass can not access the default field.




