
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

### 6. function
Okay, here it come.s Function is the most difficult part to understand in Java developers' eyes. But it actually is not that hard. Just look at them a little more times, you would get used to it. 

| java | `String memo = "memo.txt";  String extension = memo.substring(5); `         |
|------|-----------------------------------------------------------------------------|
| OC   | `NSString* memo = @"memo.txt";  NSString* extension = [memo substring: 5];` |

In short, function call is a brackets way. The grammar is just like:
`[object method:arg1 arg2Name:arg2 arg3Name:arg3`. Hahaha, I know it's hard to accept it at once, but read it a couple of times, trust me, you will get used to it. 

