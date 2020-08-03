
## Introduction
I have been an Android developer for nearly ten years so far, and I started to use React Native since 2018. Learning iOS is kind of a new thing for me since 2020. All begin from the fact that I need to develop some native module for both Android and iOS. So I have to write the iOS module as well, therefore learn iOS.

Through my process of learning iOS, things actually is kind of fun for me, however I do find out some major difference between Android and iOS that might slow you down. So I want to write all these things down, to help you get to know iOS if you intented to do so.


## iOS 101
### 1. Swift or Objective-C
Defintely Swift, if you are not a fan of C-family(C, C++, Objective-C). Swift is kind like Java, Kotlin, Ruby, a more modern language. By "mordern", I mean "no pointer", "automatic GC", "OO", and other features. 

We do all know pointer is a powerful weapon, and also a mine that might blow you off. And it's quite annoying to release point by yourselves (yes, we forget to do so occasionally!). And OO would give us many powerful stuff, like subclass, overriding, interface, and so on. 

However, if you are a React Native developer, you might need to pick up Objective-C, as the iOS code within React Native is writen in Objective-C. Some features are not mature in Swfit. Crypto is one example. Objective-C has powerful crypto API, which Swift does not have until 2020 February. Even so, Swift crypto is not mature and has not too much document by now. So you might need to know Objective-C.

One more thing to say is you may need to know Objective-C if you really want to be an expert on iOS. After all, nearly every Swift API is actually Objective-C API under the hood. That says, you might need to understand Objective-C to solve some difficult issues. And if you want to take advantage of other C library, you still need Objective-C. (Yes, Objective-C could just use C library. Android developer is not that lucky and they need to use JNI)

### 2. 


