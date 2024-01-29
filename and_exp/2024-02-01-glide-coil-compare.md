Recently we have to add cross fade to the image loading, and our project is using Picasso AND Glide. So this is a a perfect time for me to remove Picasso and use Glide, or Coil.

Now I have to decide, which one is the best image loader framework for our project, Picasso, Glide, Fresco, Coil?

# Picasso, Fresco
I have used Picasso for a long time, mostly because of the `Square` company's good fame. But also I've faced several bugs that image can't load property when I use Picasso. Once I switch to Glide, the bugs just go away. So Picasso is not a good choice for me.

And about Fresco, I was told by someone who used it that it's heavy, it's using its own ImageView (which means I have to change a lot of code), and also it's copyright might be a trouble for us in the future. So Fresco is passed as well.

# Glide Or Coil?
Now the only two player in the court is Glide and Coil. 
First, the conclusion is : they are both good. But they both have their own cons and props. 
I will list all the pros and cons, and you can choose whatever impress you most. 

## 1. lifecycle
Glide and Coil both will automatically remove the subscription/disposable for us dev. So they are both good. 


## 2. crossfade
They both support crossfade. And:
* Coil's default transition is crossfade. And the fade duration is 100ms
* Glide v3 is using crossfade by default as well. The fade duration is 300ms
* Glide v4 has removed the default transition. Now "no transiont" is the default setting.
  

```kolitn
// img1, img2 are both the https image url

// Glide
    Glide.with(this)
        .load(img1)
        .transition(withCrossFade())
        .into(ivSmall2)

//Coild
      ivSmall2.load(img1) {
          crossfade(1000)
      }

```

## 3. load svg and webp

|       | SVG | WEBP |
|-------|-----|------|
| Glide | x   | ✔    |
| Coil  | ✔   | ✔    |
 
However, Glide does not support SVG, but you can add some more config to add the SVG support. The demo is [Glide's SVG Demo](https://github.com/bumptech/glide/tree/master/samples/svg/src/main/java/com/bumptech/glide/samples/svg).


```kolitn
// Coil
    fun ImageView.loadSvgUrl(url: String) {
        val imageLoader = ImageLoader.Builder(context)
            .components { add(SvgDecoder.Factory()) } // this is the key point
            .build()
        val request = ImageRequest.Builder(context)
            .data(url)
            .target(this)
            .build()
        imageLoader.enqueue(request)
    }

```
 ## 4. preload

```kolitn
// Glide
 Glide.with(this).load(url).preload()

//Coil
  // truth is: when I use size(), there is still falsh, which is not like preloading
  val request = ImageRequest.Builder(this)
      .data(url)
      //.size(ViewSizeResolver(rv.getChildAt(0)))
      .build()
  this.imageLoader.enqueue(request)
```

p.s. The Glide has another library that is specifily for RecyclerView. However when I try it, I found out this library is not necessary to add. The Glide-Core is enough for us.

```gradle
implementation ("com.github.bumptech.glide:recyclerview-integration:4.14.2") {
  // Excludes the support library because it's already included by Glide.
  transitive = false
}
```


## 5. transition

```kolitn
// Glide
      // from anim resource ID
      Glide.with(this).load(img1)
          .transition(GenericTransitionOptions.with(R.anim.scale_in))
          .into(ivSmall1)

      // from Animator
      val anim2 = ViewPropertyTransition.Animator { view ->
          val fade = ObjectAnimator.ofFloat(view, "alpha", 0f, 1f)
          fade.duration = 2000
          fade.start()
      }
      Glide.with(this).load(img2)
          .transition(GenericTransitionOptions.with(anim2))
          .into(ivSmall2)

//Coil
    // I found out that it's kind of hard to custom transition for Coil after I reading the CorssfadeTransition class of Coil.

```
 
## 6. transform
Both Glide and Coil support the RoundedCorner and Circle shape of image. 
To custom your own transform, they require same effort to make a custom transform, such as blur. 

```kolitn
// Glide
            Glide.with(this).load(img4)
                .transform(CircleCrop())
                .into(ivSmall2)

//Coil
           ivSmall1.load(img2) {
                transformations(CircleCropTransformation())
            }
```

Here are the two famous extension libraries for Glide and Coil when you need more transformation: 
* [glide-transformations](https://github.com/wasabeef/glide-transformations)
* [coil-transformations](https://github.com/Commit451/coil-transformations)

## 7. listen

```kolitn
// Glide
    Glide.with(this)
      .load(img1)
      .listener(object : RequestListener<Drawable>{
          override fun onLoadFailed(....): Boolean {
              return false // return false to make Glide to load the error resource
          }
          override fun onResourceReady(...): Boolean {
              return false //return false to make Glide to load the image
          }
      })
      .into(ivSmall3)

// Coil -- approach 1
    ivSmall1.load(img2) {
        listener(onError = { req, _ -> println("szww coil1 error") },
            onSuccess = { req, result -> println("szww coil1 succ") },
            onCancel = {}, onStart = {})
    }

// Coil -- appraoch 2
    val req3 = ImageRequest.Builder(this).data(img2)
        .target(
            onSuccess = { resultDrawable -> ivSmall3.setImageDrawable(resultDrawable) },
            onError = { errorDrawable -> println("szww fail3") }
        )
        .build()
    this.imageLoader.enqueue(req3)
```

# Conclusion
Coil太年轻了; 和RV结合是否好? ;  v3.0还在alpha(用2.x是不是过老);   文档少而不如Glide详细 
但支持SVG, 配置不用annotation;  

In conclusion. Pros of Coil are: 
* Coil is very good, at least as good as Glide
* Coil has less method count than Glide, if you already are using OkHttp
* Coil support SVG 
* Coil's configuration does not need annotation, comparing to Glide. 

Cons of Coil are: 
* Coil v3.0 is in its alpha. If I switch to Coil now, I may have to upgrade from 2.x to v3.x again. This is a trouble I don't want to go through.
* Coil has less doc and some details are not explained as well as Glide. Such as the crossfade. Both Glide and Coil can't show crossfade if the image is in the memory. But Glide made it clear in its doc, and Coil does not. 
* It's kind of hard to make a custom transition for Coil.


Since we are using Glide already, and I found out Glide is quite good as Coil. So our project prefers Glide for now.