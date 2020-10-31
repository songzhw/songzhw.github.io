# I. Introduction

React Native, by its nature, is not a complete cross-platform framework. It is only a UI framewrok, which means you have to sink your logic to native side if you need to something complex, such as location, camera, permission requst, and so on. 

Even from UI's perspective, React Native ("RN" for short) is not powerful as well. RN itself is using facebook's yoga library to draw differnt UI element (TextView/UILabel, Button/UIButton, ...). This says, RN can not draw any shape as we wish. If you would like to draw a circle, or, more complexer, a path the same way as where you call `canvas.drawPath(path, paint)` on Android, you would feel deeply disappointed. 

I not pointing fingers right here. Facebook clearly has know this situation, and they did some hard work to improve this. One such library they made is [react-native-art](https://github.com/react-native-art/art), which could enable us to draw some shapes on RN. However, there are not much document about this library. And based on the star and fork number, clearly devs are not taking it seriously. 

|   | star | fork |
|-|-:|-|
| rn-webview | 3.2K | 1.6K |
| rn-svg | 4.9K | 666 |
| rn-art | 198 | 43 |