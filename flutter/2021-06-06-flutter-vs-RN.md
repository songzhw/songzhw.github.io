First of all, I have to say I do prefer Flutter. But this kind of preference is only my own preference, and it is not to say that React Native is bad. They are like knife/fork v.s. chopstick. I just like the chopstick more. And in this post, I'm gonna to list all the reasons that why I prefer the Flutter.

## 1. Ability of drawing UI
  Flutter is better in this area. 

  React Native, as a UI framework, is kind of odd that it does not support drawing custom views. We do have react-native-svg to help you to draw something, but to be honest, this is only a third-part library. 

  Flutter, on the other hand, could help you to draw custom views. With the help of `CustomPaint` and `CustomPainter`, you could leverage the `Canvas` class to draw anything for you -- yes, it's like the `Canvas` on Android. If you are interested, [This article](https://blog.codemagic.io/flutter-custom-painter/) could help to learn more about this ability.

  points: Flutter (1) -- React Native (0.5)

## 2. Drawing efficiency
  Flutter is using its own graphic engine, which means you don't need to pass value from RN to native side back and forth. This is even better when you are instruct native side to do an animation. 

  Of course, Flutter's own graphic engine could not reuse your existing `View`/`UIView` efficiently, because the flutter graphic engine has a hard time to communicate with your existing View,  which is written by Kotlin/Java or Swift/Objective-C. This is a downside of Flutter

  points: Flutter (1.5) -- React Native (1)

## 3. Backward compatible
  Since the release, React Native has changed too many times that does not compatible old versions, such as `mix-in`, `ReactNative.createClass({...})`. And recently Facebooks is refactoring the whole underground system, and also is refactoring the native module, which means we need to change our code if we want to use the latest React Native version. This, to be honest, is kind of horrible. You need to worry about your code every time RN releases a new version. I strongly don't like this.

  Flutter, in this area, is much better. It is still using Dart. Both Dart and Flutter are stable, even they are supporting Web and Desktop, but you still could run your existing code without changing a tiny thing. 

  points: Flutter(2.5) -- React Native (1)

## 4. One set of code
  Last time one JS dev told me that he hats the class in the JS, he just like the function. I guess this might be one of the reason why Facebook is suggesting us to use react hooks. But to be honest, when you use useEffect(fn, []), you are using class's constrcutor and destructor. This is a thought of class, not function. Function programming might be good for those calculation scenarios, as it is clean, but I doubt if function programming is good enough for a UI scenario. 

  Also, the react hook actually does not helps us a lot in the code, you still could use class to get the same effect of hooks. Besides, our code now filled with `useCallback`, `useMemo`, it is really kind of ugly to read such code.

  Functional component and class component, makes us need to learn two set of code to run React Native. And it is not over. You might need to learn `CommonJS` to config your jest/webpack/node.js scripts. 

  For Flutter, you only have one set of code: Dart. Thank goodness.

  Points: Flutter(3.5) -- React Native (1)

## 5. third-party library
  For React Native, you have to google, to find out what you need. And node_modules is a huge and heavey folder for every js project.

  For Flutter, the https://pub.dev is very easy to use. And you could see the score and coverage of every third-party library, hence you could easily get a better one for you to use. 

  Points: Flutter(4.5) -- React Native (1.5)

## 6. script  
  It's super easy to write script for your own project in React Native. 

  Unlike the `package.json` in RN, Flutter has no `script` section in the `pubspec.yaml`

  Points: Flutter(4.5) -- React Native(2.5)

## 7. language
  Flutter has more control over Dart. On the other hand, Facebook nearly have no control of TypeScript or JavaScript. 

  JavaScript does not support type, which is hard to use and read. Of course, you can use flow or TypeScript, but you may find out the next company you work might use a different language.

  The evolution of JavaScript is really slow, but the Dart is growing fast. 

  To use new version of JS, or to understand the common js style code, you might need to config babel in RN. If you are using TypeScript, you will also need to config TypeScript.  But in Flutter, you don't need to config your language. 

  Points: Flutter(5.5) -- React Native(2.5)

## Conclusion
  To be short, Flutter is more easy to use, more stable to develop, and more bright in the future, in my humble opinion. Hope this post help you a little bit if you are concerned which tech stack to choose.  