# Scratch Card View

A scratchcard is a small card, where one or more areas contain concealed information which can be revealed by scratching off an opaque covering. Applications include cards sold for gambling (especially lottery games and quizzes), or free-of-charge cards for quizzes, and to conceal confidential information such as PINs for telephone calling cards and other prepaid services.

In some app, Scratch Card effect is used to conceal some lottery information.

The view is like this : 

![](/imgs/Scratch001.jpg)
![](/imgs/Scratch002.jpg)
![](/imgs/Scratch003.jpg)
![](/imgs/Scratch004.jpg)

The key to this view is **PorterDuff.Mode** . 

## 1. PorterDuff.Mode

### 1.1 introduction of PorterDuff.Mode

"PorterDuff" : It is said that the word "PorterDuff" comes from two people, "Tomas Porter" and "Tom Duff", who coind the concept.  If you are familiar with Photoshop, the concept of "porterDuff" is like the "layer" in Photoshop. 

It is similar with puting one picture(A) on top of another picture(B). In one sepecific pixel, we can only show A's pixel, or show B's pixel, or a combined result of A and B's pixel. So this combination can bring us some useful feature to show different images. 


I stronly recommend you to read this post : http://ssp.impulsetrain.com/porterduff.html. <br/>
It expains "PorterDuff" very well, along with a specific example. 


### 1.2 PorterDuff.Mode Graphics
The graphics about PorterDuff.Mode is: 

![](/imgs/porter_duff.png)

If you read the previous post that I strongly recommended, then you will have no trouble to get this picture.

### 1.3 PorterDuff.Mode.clear
The PorterDuff.Mode.CLEAR is like a eraser to a oil painting. 

For example:

1. image A is on top of image B. Both A and B are nontransparent.<br/>
So, we can only see the picture A.

2. A draws a circle with a paint whose mode is PorterDuff.Mode.CLEAR.
Then it's like I use an eraser to erase a circle area. 
So what we see is a picture of A , and a circle part of B.<br/>
(songzhw: Yes, this is how you draw a circle portrait.)

With this PorterDuff.Mode.CLEAR, we now can draw a ScratchCard View. 


## 2. How to draw a View?

### 2.1 init the paint and other variables
It's not recommended that init objects in the **onDraw** function. So we need to initialize these variable in the constructor function.

```java

        mPath = new Path();
        mBitmapPaint = new Paint(Paint.DITHER_FLAG);

        mPaint = new Paint();
        mPaint.setStyle(Paint.Style.STROKE);
        mPaint.setStrokeJoin(Paint.Join.ROUND);
        mPaint.setStrokeCap(Paint.Cap.ROUND);
        mPaint.setStrokeWidth(150);
        mPaint.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.CLEAR));
```


### 2.2 get the width and height of a View
we all know, onMeasure() will measure the width and height of a view and its child view. But we do not know the exact width and height in the onMeasure() function. 

Then when do we know the exact width? The answer is the **onLayout()** or **onSizeChanged()** method. They both are called after the onMeasure() method, so they now can know the exact width and height.

So we now need to draw a empty bitmap whose width and height is the same width and height of this ScratchCardView in **onSizeChanged()**. This bitmap is full of a gray color. We later can erase this grey mask bitmap to get the "scratch card" UI. 

The code is :

```java
    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        mBitmap = Bitmap.createBitmap(w, h, Bitmap.Config.ARGB_8888);
        mBitmapCanvas = new Canvas(mBitmap);
        mBitmapCanvas.drawColor(0xFFAAAAAA);//draw the gray background 
    }
```

### 2.3 draw the picture
we first draw the original picture, which is a forest picture in our example. <br/>
Then we draw the previous gray bitmap on top of the original picture. So we only see the gray mask.<br/>

```java
    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        // commit the path to our offscreen. When finger are up, the path will be saved.
        mBitmapCanvas.drawPath(mPath, mPaint);

        canvas.drawBitmap(mBitmap, 0, 0, mBitmapPaint);
    }
```

## 3.Touch Event
onTouchEvent() is in charge of the touch event. I may introduce the whole touch event transfer process later. But here, you only have to know that you can deal with the touch event (touchDown, touchMove, touchUp) in this method. 

We draw a path and call ```invalidate()``` when we move our finger. When we move up our finger, the path is finished. Since we use ```mBitmapCanvas.drawPath( mPath, mPaint) ``` in onDraw(), the view will have a erasing UI when we move our finger.

```java
    @Override
    public boolean onTouchEvent(MotionEvent event) {
        float x = event.getX();
        float y = event.getY();
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                touch_start(x, y);
                invalidate();
                break;
            case MotionEvent.ACTION_MOVE:
                touch_move(x, y);
                invalidate();
                break;
            case MotionEvent.ACTION_UP:
                touch_up();
                invalidate();
                break;
        }
        return true;
    }



    private void touch_start(float x, float y) {
        mPath.reset();
        mPath.moveTo(x, y);
        mX = x;
        mY = y;
    }

    private void touch_move(float x, float y) {
        float dx = Math.abs(x - mX);
        float dy = Math.abs(y - mY);
        if (dx >= TOUCH_TOLERANCE || dy >= TOUCH_TOLERANCE) {
            mPath.quadTo(mX, mY, (x + mX) / 2, (y + mY) / 2);//二次方曲线
            mX = x;
            mY = y;
        }
    }

    private void touch_up() {
        mPath.lineTo(mX, mY);//when omit this line, this will still work!
        // kill this so we don't double draw
        mPath.reset();
    }

```



## 4. the whole codes
The whole codes is here : 
https://github.com/songzhw/SixUiViews



## reference 
http://ssp.impulsetrain.com/porterduff.html



