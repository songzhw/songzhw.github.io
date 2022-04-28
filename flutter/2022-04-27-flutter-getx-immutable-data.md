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

However, I found out I find an issue with Getx. I am making a shopping Cart class, and natually it has such model classes as below:
```dart
class FoodItem {
    final Food food;
    int quantity;
}

class CartViewModel {
    RxList<FoodItem> cart = <FoodItem> [].obs;
    
    void add(food) {
        final item = find(food);
        item.quantity++;
    }
}
```

* step 1.  Every time I click a beef (to add it to the cart) for the first time, the cart would show "beef: 1". That's correct.
* step 2.  But if I continue clicking the beef three times, and the cart should have refreshed. The cart always shows "beef: 1". Natually it should show "beef: 4".
* step 3.  Now I click another food, let's say apple, for the first time, then the cart shows correctly: "beef: 4, apple: 1"


## 4. Solution to fix the issue
Since our ViewModel class is hosting a `RxList`, which means it should have notified widgets about the item change. 
Combined step 2 and step 3 together, then I realized that the RxList would notify listener only when it has new items. If you just modify the existing items, the RxList has no changes. 

Now I can understand why. The RxList just a container of a bunch of memory address. If the address is not changed, just the content of the memory address (or, you can call it "pointer" in C language), Getx does not know there is a chagne undergoing. 


It dawned on me that this is exactly why React Redux would ask us to use `immutable data`. Oh, the Getx, just like React Redux, is a notifier system, and it should have the immutable data as well. 

Finally the solution is just so obvious: `to use immutable data`. Here is what I did, and I succeed refrsehing the page with this solution.
```dart
// instead of use `item.quantity++`, we now change to use this function:
  void changeCount(FoodItem item, int change) { // change can be 1, or -1
    final index = cart.indexOf(item); // save the oreder of items
    cart.remove(item);
    final newItem = FoodItem(food: item.food, quantity: (item.quantity + change));
    cart.insert(index, newItem); // add(item), insert(index, item)
  }    
```

## 5. Conclusion
1. Apparently the experience of React Redux helped me out of this trouble
So it would be nice to know different tech stack. It might help you sometimes

2. The Getx should save `immutable data` as a `Rx<**>` value, especially for the object type.