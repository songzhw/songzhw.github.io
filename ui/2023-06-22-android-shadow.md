Android shadow is always a pain in the axx and it never get easy for us. This is an article to help us to get to know how to make a shadow your UX designed.

# how to make a shadow in Figma
Let's put yourself in UX's shoes first. To make a shaodw, the UX could define the shadow parameters such as `offsetX, offsetY, radius, spread, color` :

![](./_image/image-20230722224021-vnn0jmo.png)

Now let's go back to the Android world. Although CSS and iOS are supporting all these shadow parameters, Android is the exception. You can't define shadow's x, y, or radius. So what should we do, to implement the UX asked?

# Theory of android shaodw
Android 5, or Material Design, has import a new concept of shadows. And we are not discuss the OS below Android 5 as they are just too rare nowasdays. 

In the Material Design, the core concept of shadow comes from lights. Just like a far-awary light and cast the light to your view: 

![](./_image/image-20230722224748-9ggenjt.png)

For this light, we would generate two kinds of shadow: spot shadow and ambient shadow.  And the blue one is the spot shadow, which is more darker, whereas the red one is the ambient shadow, which is lighter. 

![](./_image/image-20230722224930-b4gk4j4.png) 


And the life told us if we lift the view or lower the view, the shadow will change accordingly. That's true to Android world as well. The height of view is known as `elevation`.

![](./_image/image-20230722225734-3npmcan.png) 



# Practice of android shadow
Now we're finally get over the boring theory, It's time to practice. 

There are three things that can affect the shadow, which are: `spotShadow (color + alpha)`, `ambientShadow (color + alpha)`, and `elevation`. 

![](./_image/image-20230722225125-tudmjo6.png) 


And here is a list of API that we can change for shadows:

- Since Api 21
	android:elevation (set in the view)
	android:ambientShadowAlpha (set in the theme)
	android:spotShadowAlpha  (set in the theme)

- Since Api 28
	android:ambientShadowColor (set in the view)
	android:spotShadowColor  (set in the view)
	
With these five apis, we covered all you can do in the android world to generate shadows. 
However, real life are more complicated than pure theory. Let's see what pitfalls we can encouter in the real life. 	
	
# Pitfalls

We said this before, 'Just like a far-awary light and cast the light to your view, then we have shadow', To be more accurate, this saying is not 100% true. Because the shadow is not generated by the color, and it is more like generated by the view's backgroud. 

## pitfall 1: you need a bg
If you are curious why you set `android:elevation = 32dp` and there is no shadow at all. It's most likely that your view does not have a background. The code below can generate a shadow for you, in general. 

```xml
<SomeView
	android:background="@drawable/some_bg"
	android:elevation="24dp"
	/>
```


## pitfall 2: the bg can't be a svg bg
The reason why we say 'in general' is it's not 100% working all the time. It has a precondition, that your bg is a color, a png, or a `<shape>` bg. The only thing that does not work is a SVG bg. If your bg is a `<vector>` then you wouldn't have a shadow as well.


## pitfall 3: Do I need more extra space for shadow?
No, you don't have to. Android will take care of shadow. That's to say, if your view is 100dp width and 100dp height, and your shadow are like 50x50, you don't have to make your view as 150x150.  100x100 is enough for us. 

# More to come

## custom shadow shape
The reason why you have to have a bg first to make sure you would definitely have a shadow is the outlineProvider of view. View has a outlineProvider, and the shadow, to be honest, comes from this outline Provider. 


And the default outline provider is the background, that's why you have to have a bg first to make sure you can have a shadow. The outline provider values are : 

![](./_image/image-20230722233534-f6fvo8p.png)


Of course, you can customize a shadow shape by creating your own outline provider. For example: 
```kotlin
        outlineProvider = MyShadowOutlineProvider(14f.dpToPx(), 1f, 1f, 0)
        btnShadowDemo.outlineProvider = outlineProvider
        btnShadowDemo.elevation = 30f
```

And the outline provider is: 
```kotlin
class MyShadowOutlineProvider(
    val cornerRadius: Float = 0f,
    var scaleX: Float = 1f,
    var scaleY: Float = 1f,
    var yShift: Int = 0
) : ViewOutlineProvider() {

    private val rect: Rect = Rect()

    override fun getOutline(view: View?, outline: Outline?) {
        view?.background?.copyBounds(rect)
        rect.scale(scaleX, scaleY)
        rect.offset(0, yShift)
        outline?.setRoundRect(rect, cornerRadius)
    }

```        
Quite simple, right. Just a Rect, we can do the scale, and offsetX, offsetY go adjust the shadow. 


# Best practise

## theme and view
Theme just set the alpha to 1, and view can set differnt shaodw color (with alpha). This would make sure you can dynamically change the shadow. 

```xml
    <style name="Theme.Advanced23" parent="Theme.MaterialComponents.DayNight.NoActionBar">
        <item name="android:ambientShadowAlpha">1</item>
        <item name="android:spotShadowAlpha">1</item>      
		... ...
    </style>
```


Before that, you set `setSpotShadowColor(0x000000)`, now you should add the alpha to the color, which is : `setSpotShadowColor(0x88000000)`


## set shadow
If you don't have to custom shadow shape, just add a non-svg bg to your view, and add elevation. This is enough.

If you want to custom shadpw shape, you can customize your own OutlineProvider. 





	
