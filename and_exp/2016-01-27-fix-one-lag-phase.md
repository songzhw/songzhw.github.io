#An Example of Optimizing the View


## A custom view

3dTagCloudAndroid View is a View which implements a 3D Ball in Android. This view is great.<br/>
(3dTagCloudAndroid  https://github.com/misakuo/3dTagCloudAndroid )

The system works like this:
* onMeasure() : calculate the children's width and height
* onLayout() : layout all the children
* A runnable : to make the 3D ball spin automatically.  (It actually re-calculate and re-layout all the children view, and do it again N ms later.)

## Problem
Since there is a unstoppable thread running in the background, doing the calculation and the layout job, I wondered whether its efficiency was okay. So I turned on the "Profile GPU Rendering" on the phone,
and found out it was actually a little heavy for the phone. Some frames were clearly over 16ms (the horizontal green line stands for 16ms)

![](/imgs/3D_Ball_Gpu_Overweight.jpg)


## Analyzation

### Locating the problem
I decide to trace the execution of this View. Later, according to the log in the console, I found out that the onMeasure(), onLayout() are called every dozens of millisecond.

```
01-27 16:24:06.805 onMeasure()
01-27 16:24:06.815 onLayout()
01-27 16:24:06.856 onMeasure()
01-27 16:24:06.864 onLayout()
01-27 16:24:06.902 onMeasure()
01-27 16:24:06.914 onLayout()
... ...
```

I get that the onLayout() is called every dozens of millisecond, because the 3dView need to re-layout all its children view.
By the way, it's onLayout() code is this:

```java
    @Override
    protected void onLayout(boolean changed, int l, int t, int r, int b) {
        // relayout the child
        for (int i = 0; i < getChildCount(); i++) {

            View child = getChildAt(i);
            if (child.getVisibility() != GONE) {
                Tag tag = mTagCloud.get(i);  // tag saved some information that our child view needs
                tagsAdapter.onThemeColorChanged(child,tag.getColor()); 

                child.setScaleX(tag.getScale());
                child.setScaleY(tag.getScale());

                int left, top;
                left = (int) (centerX + tag.getLoc2DX()) - child.getMeasuredWidth() / 2;
                top = (int) (centerY + tag.getLoc2DY()) - child.getMeasuredHeight() / 2;

                child.layout(left, top, left + child.getMeasuredWidth(), top + child.getMeasuredHeight()); 
            }

        }
    }
```

However, why the onMeasure() function is called all the time? We actually do not need re-measure 3dView's width and height.
One measure process is enough. So, I can figure it out why the onMeasure() is called all the time and fix it.


### why the problem happens

Because the onMeasure() is called every dozens of milliseconds, so the onMeasure() must be called directly or indirectly by the Runnable.

And it is. The calling hirarchy is like this:
> 3dTagCloudView
>> Runnable. run()
>>> updateChild()
>>>> this.requestLayout()

Oh, something is not right. ```requestLayout()```?

When one view's ```requestLayout()``` is called, this view will tell its parent view to re-measure and re-layout itself.  Yes, this is why the onMeasure() is called all the time.

View's lifecycle is like this:

![](/imgs/3D_Ball_requestLayout.jpg)

So now we know how to fix it.


**Reference** : <br/>
https://plus.google.com/+ArpitMathur/posts/cT1EuBbxEgN
Arpit mathur's google+




## Improvement



## Result
Before                  |  After
:-------------------------:|:-------------------------:
![](/imgs/3D_Ball_Gpu_Overweight.jpg)  |  ![](/imgs/3D_Ball_Gpu_improve_01.jpg)


