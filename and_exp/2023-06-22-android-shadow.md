Android shadow is always a pain in the axx and it never get easy for us. This is an article to help us to get to know how to make a shadow your UX designed.

# how to make a shadow in Figma
Let's put yourself in UX's shoes first. To make a shaodw, the UX could define the shadow parameters such as `offsetX, offsetY, radius, spread, color` :

![](/_image/image-20230722224021-vnn0jmo.png)

Now let's go back to the Android world. Although CSS and iOS are supporting all these shadow parameters, Android is the exception. You can't define shadow's x, y, or radius. So what should we do, to implement the UX asked?

## Theory of android shaodw
Android 5, or Material Design, has import a new concept of shadows. And we are not discuss the OS below Android 5 as they are just too rare nowasdays. 

In the Material Design, the core concept of shadow comes from lights. Just like a far-awary light and cast the light to your view: 

![](/_image/image-20230722224748-9ggenjt.png)

For this light, we would generate two kinds of shadow: spot shadow and ambient shadow.  And the blue one is the spot shadow, which is more darker, whereas the red one is the ambient shadow, which is lighter. 

![](/_image/image-20230722224930-b4gk4j4.png) 


And the life told us if we lift the view or lower the view, the shadow will change accordingly. That's true to Android world as well. The height of view is known as `elevation`.

![](/_image/image-20230722225734-3npmcan.png) 



## Practice of android shadow
Now we're finally get over the boring theory, It's time to practice. 

There are three things that can affect the shadow, which are: `spotShadow (color + alpha)`, `ambientShadow (color + alpha)`, and `elevation`. 

![](/_image/image-20230722225125-tudmjo6.png) 


And here is a list of API that we can change for shadows:

- Since Api 21
	android:elevation (set in the view)
	android:ambientShadowAlpha (set in the theme)
	android:spotShadowAlpha  (set in the theme)

- Since Api 28
	android:ambientShadowColor (set in the view)
	android:spotShadowColor  (set in the view)