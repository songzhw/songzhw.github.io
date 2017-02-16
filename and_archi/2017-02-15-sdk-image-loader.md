### Background
I've developed a payment SDK at the previous company. Our payment SDK is easy to import, but I think there is still one issue about our SDK: our SDK only supports loading the local images. 

The main reason about it is becuase I want our SDK easy to import, and enough lightweight. If my SDK add a ImageLoader library, let's say, picasso, and all the apps that import my SDK need to import picasso, too. And this hardset rule is a little unfair for our client. All the apps that import my SDK are my clients. I want my cients happy and feel comfortable to use my SDK. So I have to resolve this problem.

### Solution
Some one normal day, I was eating my lunch, and it's dawn on me that I could use the simplest architecture principle to resolve it: **Interface-oriented programming**.

My SDK need to load image in the back-end. But do I really care what image loader library I am using? The answer is obvious: No! 

So here is what I did. 

#### step1. create a interface

```java
public interface IImageLoader(){
    public void loadImage(Context ctx, String imageUrl, ImageView iv);
    public void loadBackground(Context ctx, String imageUrl, View view);
}
```

#### step2. client app define their own image loader

If one of my client (that imports our SDK) is using Picasso, what he should do is :

```java
@Override
public void loadImage(Context ctx, String url, ImageView iv) {
    picasso.with(ctx)
            .load(url)
            .into(iv);
}
```

And if another client is still using UniversalImageLoader, that's okay. He should define the image loader this way:

```java
@Override
public void loadImage(Context ctx, String url, ImageView iv) {
    imageLoader.displayImage(url, iv);
}
```

When clients want to start our SDK to pay, he need to change a little bit
The old way is : `PaySDK.startPay(arg1, arg2);`
Now the new way is : 
```java
PaySDK.imageLoader = new PicassoImageLoader();
PaySDK.startPay(arg1, arg2);
```


#### step3. use the image loader in my SDK
when my SDK want to loader a picture, e.g a logo of one bank, all I have to do is just this:

```java
public IImageLoader imageLoader;

public void showImage(Context ctx, String url, ImageView iv){
    if(imageLoader != null){
        imageLoader.loadImage(ctx, url, iv);
    } else {
        // display the default image (placeHolder)
    }
}

```


The solution is very lightweight, and yet successful. And it avoid to import duplicated image loader library. 