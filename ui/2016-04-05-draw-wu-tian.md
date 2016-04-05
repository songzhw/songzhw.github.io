# How to draw Takenaka's flag?

If you are a fan of Japan history, you would heard of the Takenaka(Chinese and Japanese : "武田氏").  Takenaka Shingen("武田信玄") is a famous daimyo who is good at battle and cavalry. Today I want to talk about how to draw Takenaka's flag. 

Takenaka's flag is like this:

![](/imgs/Takenaka.jpg)


## 1. A normal drawing

```java
public class FlagView extends View{
	
	@Override
	public void onDraw(Canvas canvas){
		canvas.drawRect(x11, y11, x21, y21);
		canvas.drawRect(x12, y12, x22, y22);
		canvas.drawRect(x13, y13, x23, y23);
		canvas.drawRect(x14, y14, x24, y24);
	}
}
```
The calculation of x11 or y11 is really difficult and complex. It may be involve sin, cos, divide, and end up like this ``` x11 = x0 + Math.sin(45) * halfRadius ```. And all the 16 points of the four square needs to be calculated. 

It's a tough job. You can do it, but not recommended to do it. Later, I will introduce a smart way.


## 2. A smarter drawing


![](/imgs/Takenaka_mine.jpg)
