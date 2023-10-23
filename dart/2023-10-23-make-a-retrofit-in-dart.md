[Retrofit](https://square.github.io/retrofit/) is a very powerful library in Android. It can achieve many functionality without dev writing a bunch of redundant code. Also the code in Retrofit is very easy to read, thus very easy to maintain as well since all the config of one endpoint are lies in a function and its annotation. This makes me wonder, do we have such a library in Flutter?

After digging the pub.dev, I did find out such a library, [retrofit-flutter](https://pub.dev/packages/retrofit). But to be honest, I am not very happy about this library, because I don't like the extra step to generate the flutter manually. Android has no such step. So I was thinking of mkaing my own one.

# 1. annotation
Annotation is something to declare some extra info of a class, a method, or a parameter. In Java, you can define one annotation by using `@interface`; In Kotlin, you can do so by using `annotation class`. 

However, in Dart, any plain class can be annotation. Such as: 
```dart
class Todo {
  final int id;
  final String name;
  const FishTodo(this.id, this.name); 
}

class Worker {
  @Todo(23, 'air fly')
  void work(bool isRepeat) {
    print('work : $isRepeat');
  }
}
```

# 2. read annotation's values
In Java, we have reflection to get the detail info of one class, method or parameter. In Dart (and Swift), such mechanism is called `mirror`. 

There are `ClassMirror`, `MethodMirror`, `ParameterMirror` ... classes in Dart to represent the mirror of a class, method or parameter. And all these mirror class has a `metadata` property to keep log of their own annotation. 

For us, to read values of an annotation, we just need to use `mirror.metadata` : 

```dart
  //use reflect(obj) the mirror object
  InstanceMirror mirror = reflect(Worker()); 
  
  // get all the methods, returns a map whose key is Symbol(like method name), and value is MethodMirror
  Map<Symbol, MethodMirror> methods = mirror.type.instanceMembers;  //mirror.type是一个ClassMirror对象
  
  // iterate over all the method, to find out the methods which are annotated by the Todo annotation
  methods.forEach( (symbol, methodMirror) {
      try {
          List<InstanceMirror> annotationList = methodMirror.metadata;
          InstanceMirror annotation = annotationList.firstWhere ( (meta) => meta.reflectee is Todo)
          Todo todo = annotation.reflectee;
          print('name = ${todo.anme}, id = ${todo.id}'); //=> name = air fly, id = 23
      } catch (err) { 
          // the firstWhere may throw errors if there is no such Todo annotation for some method
      }
  }
```

# 3. Dynamic proxy

In Java, we have `Proxy.newProxyInstance()` and `InvocationHandler` to generate a dynamic proxy. But Dart has a different mechanism.

Let's step back a little bit, to take a look how Ruby implement the dynamic proxy. Ruby has a `missing_method` method for each class, so if you don't have one method, and this `missing_method` will get called, where you can inject your own proxy in. 
```ruby
class User 
    def method_missing(name, *args) 
        ....
    end
```

Dart has a similar mechanism with Ruby, the only difference is the method name is called `noSuchMethod` in Dart.  Here is one example: 
```dart
class Face {
  @override
  dynamic noSuchMethod(Invocation invocation) {
    print('called: $invocation');
    return 23;
  }
}

main() {
    dynamic obj = Face();
    final result = obj.foo(); // there is no such `foo` method in the Face class !!!
    print("result = $result"); //=> result = 23
}
```

# 4. to make a retrofit
With the knowledge of previous bullet points, we now can make our own Retrofit. 

First, we define some annotation and some endpoint interface/class for us: 
```dart
// 定义一个annotation类
class Get {
  final String url;
  const Get(this.url);
}

class UserService {
  @Get('https://www.somesite.com/api/Gut')
  String getUser();

  @override
  dynamic noSuchMethod(Invocation ivc) {
      ... // here is the key point, which will be implemented later
  }
}


void main() {
  UserService http = UserService();
  final resp = http.getUser();
  print('resp = $resp');
}

```

And the key point is we call the Dio to send http request when we got some `getUser` call. Yes, since the `getUser` method is not implemented yet, so the `noSuchMethod` will get called when we call `http.getUser()` as well.
```dart
  @override
  dynamic noSuchMethod(Invocation ivc) {
    final objMirror = reflect(this);
    // `ivc.memberName` is the method name, which is a Symbok object 
    final methodMirror = objMirror.type.instanceMembers[ivc.memberName]!; 
    // find out the `Get` annotation
    final annoMirror = methodMirror.metadata.firstWhere( (meta) meta is Get ); 
    String url = annoMirror.reflectee.url;
    
    Future resp = dio.get(url);
    return resp; // 返回dio得到的response
  }

```
