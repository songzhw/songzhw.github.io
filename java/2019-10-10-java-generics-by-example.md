
Let's say we have an app for displaying cute pets. Naturally, we would have lists of pets. But before Java 1.5, the reality is not good for our Developer. Let's see the tough reality among Java 1.0 to Java 1.4.



\## Why we need generics?

Before we start, let me give you some context. We have three class, and their hierarchy is like this:

\```

​    Animal

​    /    \

  Cat    Dog

\```

Now let's begin.



\### 1. to reduce duplicate code

We could have `CatList` and `DogList` classes, or we could have `List<Cat>` and `List<Dog>` objects. They seems similar, except the latter share a same class, which, for us, removes duplicate code. It helps us manage our code, extend our code, and change our code easier.



\### 2. array is not a perfect option for us

You may say, we could save the pets into an array, like `Cat[]`, `Dog[]`, and `Animal[]`. Yeah, you can, but this is just not the best option for us. Why? Let's have a look at the below code:



\```java

Dog dog = new Dog();



Cat[] cats = new Cat[1];

Animal[] animals = cats;

animals[0] = dog;

\```



These code seem fine, but they are not. Yes, you will see no warning or errors when you write it (-- which makes this even more evil), but you will get a crash if you want to run it. 

![image-20191009105301907](_image/image-20191009105301907.png)

