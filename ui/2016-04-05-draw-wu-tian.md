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

### 2.1 Canvas Transformation
The answer is actually the transformation of canvas. In Andorid, you can scale, rotate, translate the canvas as you like. These transformation is good for the coordinate calculation. 

For example, If I want to draw a circle in somewhere, my codes are like this:

```java
    // drawCircle(centerX, centerY, radius, paint)
    canvas.drawCircle(100, 100, 50, paint);
```

But if I translate the canvas to the (100, 100) point, which is like you move your textbook in the real life, then drawing the same circle would be like this:

```java
	canvas.save(); // save the initial state of this canvas
	canvs.translate(100, 100);
	canvas.drawCircle(0, 0, 50, paint);
	canvas.restore(); // restore to the initial state of this canvas
```

In this way, we do eliminate a lot of coordiante calculation. 


### 2.2 Drawing Flag
Back to our job, we now can forget about all the complex coordiante calculation. We can do the same job with only simple codes.


Firstly, I abstarct one square out of the flag, because every four of them are similar. I call this class as "Cube".

```java
    class Cube {
        public Rect bounds;
        public ValueAnimator anim;
        public void draw(Canvas canvas, Paint paint) {
            canvas.drawRect(bounds, paint);
        }
    }
```

Secondly, now I start to draw the flag. The simplified codes are as follow. 


```java

public class FlageDrawable extends Drawable {

    private Cube[] children = new Cube[4];
    private Rect rect;

    // override from Drawable
    @Override
    protected void onBoundsChange(Rect bounds) {
        super.onBoundsChange(bounds);
        rect = getBounds();
        for (int i = 0; i < 4; i++) {
            children[i] = new Cube();
            children[i].bounds = new Rect(rect.left + rect.width()/6, rect.top+rect.height()/6,
                    rect.left + rect.width()/2 - 3, rect.top + rect.height()/2 - 3);
        }
    }

    // override from Drawable
    @Override
    public void draw(Canvas canvas) {
        for (int i = 0; i < 4; i++) {
            Cube cube = children[i];
            canvas.save();
            canvas.rotate(45 + i * 90, rect.centerX(), rect.centerY());
            cube.draw(canvas, paint);
            canvas.restore();
        }
    }
}

```

Now we can see that every ```Cube``` has the same boundary, but we do draw four squares, instead of one. Because our canvas is always rotating. As Canvas rotate 45, we can draw a new square.

The result is :

![](/imgs/Takenaka_mine.jpg)


