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


## obstacle #2 Complex Animation system
In the Android, one `Animation` or `Animator` class is good enough to build, and start an animation.
But in the Flutter, no, things are more complicated (than it should be, -_-).

In the fultter, you have `Tween` to control the interpolator, and you have `Animation` to calculate the value change, and you also need a `AnimationController` to start or stop an animation. At last, your AnimationController will need a `TickerProvider` to initialized. Yes, you have to provide four class before you can really start an animation. It's not a simple beauty that I enjoy, but that's the life (in Flutter). 

So we have to provide all these object before we can start an animation:
```dart
// in the state class
class _LoadingWidgetState extends State<LoadingWidget> with TickerProviderStateMixin{
  ctrl = AnimationController(vsync: this, duration: 2000); // `this` is the tickerProviderStateMixin
  anim = Tween(begin, end).animate(ctrl);
  ctrl.repeat();

  // in the drawing
  final raidus = animation.value;
  canvas.drawCircle(center, radius, paint);
```


## Obstacle #3 Flutter has no Keyframe animation
In the Android, we have Animator that supports keyframe changes. And it can be even more powerful by supporting multiple property's keyframe change. The code is someting like this: 
```kotlin
// alpha animation can be just a normal animation
val value2 = PropertyValuesHolder.ofFloat("alpha", 0f, 1f)

// rotation animation is based on keyframe
val value1 = PropertyValuesHolder.ofKeyframe("rotation",
    Keyframe.ofFloat(0f, 180f), // Keyframe.ofFloat(fraction, value)
    Keyframe.ofFloat(0.3f, 270f),
    Keyframe.ofFloat(1f, 360f),
)

val anim = ObjectAnimator.ofPropertyValuesHolder(textView, value1, value2)
anim.duration = 5000
anim.start()
```

### Staggered animations in Flutter
Unfortunately for us, Flutter has no such built-in keyframe animation support yet. The only way it seems close to it is the `Staggered animations` by using a class called `Interval`. 

```dart
    ctrl = AnimationController(duration: Duration(seconds: 5), vsync: this);
    final curve1 = Interval(0.1, 0.6); 
    anim1 = Tween<double>(begin: 1.0, end: 0.0)
        .animate(CurvedAnimation(parent: ctrl, curve: curve1))
      ..addListener(() => setState(() {}));
```

So the animation will last from 10% to 60% of the duration. This seems like a keyframe animation, and it is. But the downside is quite clear too:
1). Interval can only support exactly 3 keyframes. (0%-10%, 10%-60%, 60%-100%).
aka. if I want to have two keyframes or five keyframes in one animation, Interval does not support it. 
2). Interval can support only one type of value, and can't support multiple property animation at the same time. 

So this `Staggered Animation` seems not working in the spin-kit scenario.

### `keyframe_tweens`
Lucky for us, we have a 3rd-party library that support multiple property's keyframe animation for us. This library is so-called [keyframe_tweens](https://pub.dev/packages/keyframes_tween). 

An example of its usage is:
```dart
class _ExampleState extends State<Example> with TickerProviderStateMixin {
  late final controller = AnimationController(
    duration: const Duration(seconds: 10),
    vsync: this,
  );

  // define the animation
  final tween = KeyframesTween([
    KeyframeProperty<Size>(
      [ Size(10, 10).keyframe(0),
        Size(100, 100).keyframe(0.5, Curves.easeInOut),
        Size(200, 200).keyframe(1.0)]),
    KeyframeProperty<Color>(
      [ Colors.black.keyframe(0.0),
        Colors.red.keyframe(0.8, Curves.easeInOut),
        Colors.blue.keyframe(1.0)],
      name: 'background',
    ),
  ]);



  @override
  Widget build(BuildContext context) {
    return ValueListenableBuilder<KeyframeValue>(
      valueListenable: tween.animate(controller), // generate an animation by KeyframesTween
      builder: (context, values, _) => Container(
        width: values<Size>().width, // how to use animation value
        height: values<Size>().height, // how to use animation value
        color: values<Color>('background'),
      ),
    );
  }
}
```

## obstacle #4 flutter animation has no `delay` property
This is kind of odd, since a `delay` property in the Animation is common practice in the industry. Android has it, iOS has it, even [React Native has it](https://reactnative.dev/docs/animated#composing-animations). 

Lucky for us, it's not hard to make a delay for the start of anmation. 
```dart
  Future.delayed(
      Duration(milliseconds: (second * 1000).round()),
      ctrl.repeat(),
    );
```


If you are using `Getx`,then it would be easier since you can write this: 
```dart
  delayInSecond.delay(()=> ctrl.repeat()); // delay(fn) is from `Getx` library`
```  


## obstacle #5 flutter has no invalidate method
In the android, you can call
* `View # invalidate()` to refresh View
* or `Drawable # invalidateSelf()` to refresh Drawable

flutter is a declarative framework and has no such refresh method to call. So we have to figure out a workaround this. The solution I had in mind is to make a common animation. We don't care the value of this common animation, we just need to refresh the widget by this common animation.

```dart
class _LoadingWidgetState extends State<LoadingWidget> with TickerProviderStateMixin{
  // this animation is just to make the state to refresh
  late Animation<double>? animToRefresh;
  late AnimationController ctrlToRefresh;

  @override
  void initState() {
    super.initState();
    ctrlToRefresh = AnimationController(vsync: this, duration: 1.seconds);
    animToRefresh = Tween(begin: 0.0, end: 1.0).animate(ctrlToRefresh);
    ctrlToRefresh.repeat();
  }

  @override
  Widget build(BuildContext context) {
    if(animToRefresh == null) return Container();
    return CustomPaint(painter: LoadingPainter(sprite));
  }

}
```

## obstacle #6 canvas has no pivot 
If you can `canvas.scale()` or `canvas.rotate()` in Flutter, the scale and rotation will be transform as if the left-top corner is the center. This is not quite right for most cases. 
If we want to custom the transform pivot, we can use this extension: 

```dart
extension ScaleWithPivot on Canvas {
  void scaleWithPivot({required double sx, required double sy, required Offset pivot}) {
    this.translate(pivot.dx, pivot.dy);
    this.scale(sx, sy);
    this.translate(-pivot.dx, -pivot.dy);
  }
}

extension RotationWithPivot on Canvas {
  void rotateWithPivot({required Offset pivot, required double degree}) {
    this.translate(pivot.dx, pivot.dy);
    this.rotate(degree * pi / 180.0);
    this.translate(-pivot.dx, -pivot.dy);
  }
}
```

Then you are free to use any point as your pivot, such as you want to rotate with the center as the pivot: 
```dart
@override void paint(Canvas canvas, Size size) {
  final rect = Offset.zero & size;
  canvas.rotateWithPivot(pivot: rect.center, degree: 180)
}  
```

