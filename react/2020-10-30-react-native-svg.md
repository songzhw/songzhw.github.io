# I. Introduction

React Native, by its nature, is not a complete cross-platform framework. It is only a UI framewrok, which means you may have to sink your logic to native side if you need to something complex, such as location, camera, permission requst, and so on. 

Even from UI's perspective, React Native ("RN" for short) is not powerful as well. RN itself is using facebook's yoga library to draw differnt UI element (TextView/UILabel, Button/UIButton, ...). This says, RN can not draw any shape as we wish. If you would like to draw a circle, or, more complexer, a path the same way as where you call `canvas.drawPath(path, paint)` on Android, you would feel deeply disappointed. 

I not pointing fingers right here. Facebook clearly has know this situation, and they did some hard work to improve this. One such library they made is [react-native-art](https://github.com/react-native-art/art), which could enable us to draw some shapes on RN. However, there are not much document about this library. And based on the star and fork number, clearly devs are not taking it seriously. 

|   | star | fork |
|-|-:|-|
| rn-webview | 3.2K | 1.6K |
| rn-svg | 4.9K | 666 |
| rn-art | 198 | 43 |

Someone may ask, is it really necessary that we need to draw something on the screen?<br/>
: It depends. If your app is complex, then you might have the need to draw something for yourself. Examples are listed below, such as circle-shape Avatar, linear button and so on.

When you need to draw something, but RN's JSX apparently don't offer you a way to do that. What should you do? <br/>
: Okay, then React-Native-SVG come to the rescue. 

# II. SVG & React-Native-SVG

## 2.1 SVG
What we used often is `JEPG, PNG` images. These images has one limit: it shows jagged edge when it is scaled. 
On the other ahdn, SVG is another different type. No matter how much you scale a SVG picture, it still show the image smoothly.

Here is the picture to tell the difference :
![](images/bitmap-vs-svg.png)

SVG files are actually text files, whereas bitmap files are binary files. Here is a SVG that displays "check"(✔) icon:

```xml
<svg width="24" height="24" viewBox="0 0 24 24">
  <path d="M9 16.2 L4.8 12 l-1.4 1.4 L9 19 21 7 L-1.4 -1.4 z"/>
</svg>
```

We can see it tell the browser or any SVG's user to how to draw a path, from which point to which point. This is what SVG did. 

## 2.2 React-Native-SVG
rn-svg is another library that Facebook made to draw shapes. Actually iOS don't support SVG officially, and you have to import 3rd-party library, such as SVGKit, to draw SVG. But rn-svg still do a lot to support most of SVG attributes. 

## 2.3 Draw a circle
Let's take circle for an example. 

```xml
<Svg height="100" width="100">
  <Circle cx="50" cy="50" r="50" fill="pink" />
</Svg>
```
It could draw a cicle whose fill is pink color. 

Svg is not just that. It can draw Rectangle, Triangle, Polygon, Line, even Text and Image. 

Furthuremore, with the super power of Sketch or Adobe Illustrator, we can get complex path, and use them to generate wonderful animations.

## 2.4 Draw react native element in SVG
In general, the SVG shapes, and only them, must be named inside `<Svg>` element. You can't just directly put a `<FlatList>` inside `<Svg>`. However, if you do need to do so, we still have an approach to help you. This is the `<ForeignObject>` element.

`<ForeignObject>` is a SVG shape, and also it could contain React Native items. Here is an example of how to use it:

```xml
import {Image} from "react-native"
import Svg, {Circle, ForeignObject} from "react-native-svg"

<Svg>
	<Circle cx="50" cy="50" r="50" fill="pink" />
	<ForeignObject>
		<Image src={..} source={...}/>
	</ForeignObject>
</Svg>
```


# III. SVG in Practice