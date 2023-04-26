[Android SpinKit] is a library that shows diffent loading animations. And it has one of the most elegant, beautiful UI structures I've ever seen. Also this beautiful structure enables us to easily add a new animation for loading, aka, its extension is quite easy.  Naturally I want to copy this elegant UI structure to the Flutter world. When I did this, I found out I have many obstacle to overcome, and I managed to do so. This article is to introduce all the pitfalls or obstacle I had. 

## obstacle #1 : Flutt has no ObjectAnimator
Android has an `ObjectAnimator` which you can just animate its properties. For example, the code below could make a view to scale from original size to twice size, and go back to normal size again: 

```kotlin
// in the Sprite class (actually it is a Drawable class)
val anim = ObjectAnimator(myView, "scale", 1f, 2f, 1f)
anim.addUpdateListener { invalidateSelf() } // Drawable # invalidateSelf()
anim.start()

// in the myView class
var _scale = 1f // setter will be called by the ObjectAnimator
```

Now Flutter has no such handy animator framework. What can we do then?
: So we have to create our own animation, and treat the animation's value to "scale" by ourself. It's not that convinient as Android, but it's good enough to use.

```dart
// in the widget
animation = Tween(begin, end).animate(ctrl)
ctrl.repeat();

animation.addListener( () {
  setState( (){...})
});

// in the CustomPainter
final raidus = animation.value;
canvas.drawCircle(center, radius, paint);
```


## 
