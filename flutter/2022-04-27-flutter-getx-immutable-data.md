# Flutter, Getx, and Notif

## 1. Flutter's shortage

Flutter, just like other `Declarative UI` framework (such as React, SwiftUI, Jetpack Compose), has the same issue – `the performance is bad when it comes to a real business project`. 

The one of main reasons why the performance is bad is these declarative UI framework are all using `setState` to `refresh/re-render/re-compose` the whole page.  I just want to refresh one view, but all the view in this page got refreshed. Quite naturally, the performance of drawing is bad. 

Come back to the Flutter world, Flutter use `StatefulWidget` to host a state, and you can call `setState( (){... })` to refresh the whole page.  Let's say I have a page, which has a button and a text. Every time I click the button, the text's number would increase by 1. So the code would be:

```dart
class HomePage extends StatefulWidget {
    ...
}

class HomeState extends State<HomePage> {
    int count = 0;
    
    @override Widget build(context) {
        return Column(children: [
            Text("count = $count"),
            TextButton(onPress: ()=> setState( ()=> count++ )),
            Image(...)
        ]);
    }
}

```

The most apparent shortage of this code above are :
* It has the boilerplate code. You have to write a state class and a statefulWidget class.
* The performance is bad. The Image view is not changed, but it will get re-rendered by the clicking the button. 


## 2. Getx, come to help
However, [Getx](https://pub.dev/packages/get) could help us to reduce these unnecessary re-render.  Getx is using a reactive stream to tell a specific widget that "it's your turn to re-render". 

By "reactive", yes, you are right, just like the RxJava/RxSwift/RxDart/RxJS. GetX has some classes called "RxInt", "RxString", … And when you change the the value of these `Rx**` class, the observer will get notified automatically. 

Here is an example modified from the previous StatefulWidget example
```dart
// 1. The parent class is not `StatefulWidget` anymore
//.   Hence, the boilerplate code of creating state is gone as well
class HomePage extends StatelessWidget {
    int count = 0.obx;  // 2. the count actually is a `RxInt` type
    
    @override Widget build(context) {
        return Column(children: [
             // 3. wrap the widget with Obx, to receive the updated value automatically
            Obx(() => Text("count = $count")), 
            TextButton(onPress: ()=> count++), // 4. Yes, RxInt can use "++" as well
            Image(...)
        ]);
    }
}
```

By doing so, every time we click a button, 
* Only the Text will get re-render.  That's a big step forward !!
* No more boilerplate code of creating State class


## 3. Got an issue with Getx


## 4. Solution to fix the issue


## 5. Conclusion
