
Let's say we have an app for displaying cute pets. Naturally, we would have lists of pets. But before Java 1.5, the reality is not good for our Developer. Let's see the tough reality among Java 1.0 to Java 1.4.



##I. Why we need generics?

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



First of all, `animals` is an array of animals, so it's okay to add a dog object, and it's okay to add a bunch of cats. -- That's why Java compiler does not say it's wrong



But when we run this code, Java just noticed that "Wait a minute, when you use `animals = cats`, it actually becomes an array of Cat, so you can't add Dog into it anymore!" -- This seems fair too. 



Just it's confusing if you compare these two behavior (at the run time, and the one at the compile time).  Yes, this kind of confuse is exactly why `Array` in Java is not good for "type security".   



Now I must say one extremely important things: `Arry's type insecurity is exactly what the Java Generics wants to avoid`. List want to avoid this kind of `ArrayStoreException` at the runtime. Understand this, then you will understand many odd behavior of generics, which we'll cover later. But before we move on, please remember this again:`Arry's type insecurity is exactly what the Java Generics wants to avoid`!!! 

## II. Examples



\### 1. List<Animal>

Still the Pet App. Let's say I have a home page to show all kinds of pets. So I need a `List<Animal>`. Here is the code



\```java

List<Animal> animals = new ArrayList<>();

animals.add(new Cat());

Animal cat = animals.get(0);

\```



This is correct and usual. You may have no interest in it. Fair enough, let's go a little further.



\### 2. Common RecyclerView.Adapter

Inside the Pet App, we have a screen called `Cat List Screen`, to display the cute cats; we also have another screen, `Dog List Screen` to display the cute dogs; The UI of both screen are identical: only show a RecyclerView; the ViewHolder of Recycler is same too. Just the data is different. One is `List<Cat>, and another is `List<Dog>`. 



Naturally, I only need one Adapter, one CommonAdapter class, for both screens. In this CommonAdapter, I have a `setData(pets)` method, which I can pass in a `List<Cat>`, and I can also pass in a `List<Dog>`. According to the previous Animal[] array example, I just make the definition as `public void setData(List<Animal> animals)`, things should be good.



Unless, is it?

Let's make the `setData()` method this way, and pass in a `List<Cat>` data, what could happen?



![image-20191009140141240](_image/image-20191009140141240.png)