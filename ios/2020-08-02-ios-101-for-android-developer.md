
## Introduction
I have been an Android developer for nearly ten years so far, and I started to use React Native since 2018. Learning iOS is kind of a new thing for me since 2020. All begin from the fact that I need to develop some native module for both Android and iOS. So I have to write the iOS module as well, therefore learn iOS.

Through my process of learning iOS, things actually is kind of fun for me, however I do find out some major difference between Android and iOS that might slow you down. So I want to write all these things down, to help you get to know iOS if you intented to do so.


## iOS 101
### 1. Swift or Objective-C
Defintely Swift, if you are not a fan of C-family(C, C++, Objective-C). Swift is kind like Java, Kotlin, Ruby, a more modern language. By "mordern", I mean "no pointer", "automatic GC", "OO", and other features. 

We do all know pointer is a powerful weapon, and also a mine that might blow you off. And it's quite annoying to release point by yourselves (yes, we forget to do so occasionally!). And OO would give us many powerful stuff, like subclass, overriding, interface, and so on. 

However, if you are a React Native developer, you might need to pick up Objective-C, as the iOS code within React Native is writen in Objective-C. Some features are not mature in Swfit. Crypto is one example. Objective-C has powerful crypto API, which Swift does not have until 2020 February. Even so, Swift crypto is not mature and has not too much document by now. So you might need to know Objective-C.

One more thing to say is you may need to know Objective-C if you really want to be an expert on iOS. After all, nearly every Swift API is actually Objective-C API under the hood. That says, you might need to understand Objective-C to solve some difficult issues. And if you want to take advantage of other C library, you still need Objective-C. (Yes, Objective-C could just use C library. Android developer is not that lucky and they need to use JNI)

Swift actually is very much like Kotlin. So I don't want to waste too much time on it. I would mostly introduce Objective-C ("OC") in  this post.

### 2. types of OC
Compared with Java and JavaScript, OC's types are a little different:

|        | java         | Objective-C    | JavaScript  |
|--------|--------------|----------------|-------------|
| string | String       | NSString       | string      |
| int    | int; Integer | int; NSInteger | number      |
| map    | HashMap      | NSDictionary   | Object; Map |
| byte[] | byte[]       | NSData         | ArrayBuffer |


### 3. number
Since Objective-C is a superset of C, so OC does support types like `unsigned long`, `short`, and `unsigned int` types, which Java is not supporting anymore.

Literal number value would start with `@`, such as `@20`,which indicate this is an OC value, not a C value. Yes, OC and C are quite different from each other, and we would talk about it later.

### 4. String
String literal value is also start with `@`, to indicate this is not a C string (C string is actually a char[]). One example would be `@"hello world"`.

When you need to insert value into a NSString, just like `"hello $name"` in Kotlin, you could use `%@` as a placeholder.  `%@` indicates this is a placeholder for an OC object. 
Example:

```Objective-C
NSString* name = @"szw";
NSLog(@"hello %@", name);
```

### 5. object
Just like its name, Objective-C contains lots of objects. 

When you new an Object, the object you just initialized is actually a point that point to a piece of memory. So unlike our Java approach (`String name = "szw"`), the Object in OC should be defined as a pointer!

```Objective-C
/* Wrong */   NSString name = @"szw";
/* Correct */ NSString* name = @"szw";
```

### 6. function call
Okay, here it come.s Function is the most difficult part to understand in Java developers' eyes. But it actually is not that hard. Just look at them a little more times, you would get used to it. 

| java | `String memo = "memo.txt";  String extension = memo.substring(5); `         |
|------|-----------------------------------------------------------------------------|
| OC   | `NSString* memo = @"memo.txt";  NSString* extension = [memo substring: 5];` |

In short, function call is a brackets way. The grammar is just like:
`[object method:arg1 arg2Name:arg2 arg3Name:arg3`. Hahaha, I know it's hard to accept it at once, but read it a couple of times, trust me, you will get used to it. 

More examples are:
```objective-C
/* declaration */ + (NSData *)decryptContent:(NSData *)encrypted withKey:(NSData *)key; 
/* usage */      [obj decryptContent: encrypted withKey: key];

/* declaration */ - (void)stop;
/* usage */       [obj stop];
```
### 7. source files
Unlike a single ".java" or ".kt" to stands for a class, Class in OC would be two files: `.h` and `.m` file. 

* `.h`: It's only a declaration. Normally you would list your public method and variable here. So other class can access them.
* `.m`: It's the implementation code. Functions here is no longer just declaration, it has to have content in it. 

### 8. class
As we said before, a class would be split to two files, one .h file and one .m file.
Here is an example:

```objective-C
// MyClass.h
@interface MyClass: NSObject {
  NSString* name;  //property
}
-(void)hello:(NSString*)name; //method
@end
```
1). class declaration is a defined in `@interface`.
2). unlike java, even your class is to subclass NSObject (like Object in Java), you have to write this superclass down.
3). method declaration is a little complex:
  3.1). `-` means this is a instance method; `+` means this is a static method. 
  3.2). (void) is the return type
  3.3). name is the argument
  3.4). so we could call this function by `[obj hello:@"szw"];`

```objective-C
// MyClass.m
@implementation MyClass 
-(void) hello:(NSString*)name{
  NSLog(@"hello, %@", name);
}
@end
```
class implementation need to be wrapped inside `@implementation`, and you do whatever you want to do inside the function body.

### 9. category
Category is just like Kotlin's extension. So you could add more functionality to a class, yes, even a system class.

Defining a category is very much like defining a class. Just instead of declare its super class, you need to describe the category name by using "()"
Here is an example to extend NSString so it could have an extra method called "uuid":

```objective-c
// NSString+Uuid.h
#import <Foundation/Foundation.h>

@interface NSString (Uuid)
+ (NSString*) uuid;
@end

// NSString+Uuid.m
#import "NSString+Uuid.h"
@implementation NSString (Uuid)
+ (NSString*) uuid {
  CFUUIDRef uuidRef = CFUUIDCreate(kCFAllocatorDefault);
  CFStringRef stringRef = CFUUIDCreateString(kCFAllocatorDefault, uuidRef);
  CFRelease(uuidRef);
 
  NSString* uuidString = (NSString*)CFBridgingRelease(stringRef);
  return uuidString;
}
@end
```

### 10. protocol
Protocol is a combination of several functions. Yes, this is a objective-c version of `Interface` in Java. And it does, just like interface, only have `.h` file. No `.m` file for protocol, as the `.m` file is the implementation. And interface has no implementation.

```objective-c
@protocol ServerPlugin <NSObject>
  @required -(void) serve: (int) id;
@end

// if you want to use the protocol, use it as a generics:
@interface HttpServer: NSObject<ServerPlugin>;
@end
```

One more thing that might worth bringing up is, we could assign a object to an interface type in Java. like `IPlugin plugin1 = map.get("key1")`;
However, in objective-c, Protocol is not a type, it is more like a generics. So if you want to transform another object to some specific protocol, here is what you can do:
`NSOjbect<PluginProtocol> plugin1 = [dictionary objectForKey:@"key1"];`

### 11. memory management
You would be surprised that how annoying the memory mangement in C when you write app in C. This was the same situation you might face in OC. 

```objective-C
-(NSArray*) getArray {
  NSArray* ary = [[NSArray alloc]init];
  return ary;
}
```
Before 2011, the above code is actually quite wrong. Because you just return some local variable from a method. Once the method is done, the method stack would be dismissed. Then this local variable is gone. Then what you got is actually a wild pointer. What you need to do, is to make sure your object would survive long enough, and also make sure you free these memory space when you are done. Imagine what if you have 20,000 objects, this would be a disaster. 

Fourtunately for us, Apple introduce ARC(Auto Reference Counting), which would inject the code to manage reference number for you. In short, ARC is just like Java's GC, so you don't have to worry about the memory anymore.

However, you are not 100% away from memory management. When you try to use some Core Foundation library, which is in C and is not covered by ARC, then you would need to inject your memory management code there. 
