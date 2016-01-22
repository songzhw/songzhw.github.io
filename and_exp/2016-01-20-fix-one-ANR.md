# Log of fixing a ANR

Today, I found manolovn publish a new Open-Source library, and here is the address: https://github.com/manolovn/trianglify.

This library tries to make a simple yet beatiful geometrical graphics, and it do succeeed. 

![](/imgs/20160120_00.jpg)

## Problems
However, when I try to drag the seekbar, and the app goes frozen. Seconds later, a ANR dialog shows.

![](/imgs/20160120_01.jpg)

In the same time, the lot, lot of GC logs come out:

```
01-20 17:27:25.890 25019-25019/cn.six.open D/dalvikvm: GC_FOR_ALLOC freed 2047K, 33% free 15237K/22588K, paused 21ms, total 21ms
01-20 17:27:26.050 25019-25019/cn.six.open D/dalvikvm: GC_FOR_ALLOC freed 1914K, 32% free 15370K/22588K, paused 22ms, total 22ms
01-20 17:27:26.290 25019-25019/cn.six.open D/dalvikvm: GC_FOR_ALLOC freed 2181K, 33% free 15237K/22588K, paused 22ms, total 22ms
01-20 17:27:26.480 25019-25019/cn.six.open D/dalvikvm: GC_FOR_ALLOC freed 2047K, 33% free 15237K/22588K, paused 22ms, total 22ms
... ...
... ...
```

## Locating the problems (1)

Since it has something to do with GC, aka "memory", so I want to try to analyze the memory firstly.

I dumped java heap with Android Studio 1.5.1, and get a analized memory result here:

![](/imgs/20160120_02.jpg)

OMG, there are more than 1400 "Triangle" objects and more than 68,000 "Point" objects in the memory. Take the "Point" objects for example, they may not occupy too many memory ( 1092KB, or 1M in in this example), but the huge amount of objects are a bad signal, because the initialization and destory of objects also consume our CPU and time.

So my first target is to eliminate the huge amount of objects  in the app, which is weird too. 


I notice many many objects are generated in a short time, and the onDraw() may be draw many times. So my first guess is that something may be wrong in onDraw() .

And I got it.  The original code is:

```java
        colorGenerator.setCount(triangles.size());
        for (Triangle triangle : triangles) {
            Path path = new Path();
            path.setFillType(Path.FillType.EVEN_ODD);

            path.moveTo(triangle.a.x, triangle.a.y);
            path.lineTo(triangle.b.x, triangle.b.y);
            path.lineTo(triangle.c.x, triangle.c.y);
            path.lineTo(triangle.a.x, triangle.a.y);

            path.close();

            trianglePaint.setColor(colorGenerator.nextColor());
            canvas.drawPath(path, trianglePaint);
        }
```

Now I put the path initialization out of this function:

```java
        colorGenerator.setCount(triangles.size());
        for (Triangle triangle : triangles) {
            path.reset();

            path.moveTo(triangle.a.x, triangle.a.y);
            path.lineTo(triangle.b.x, triangle.b.y);
            path.lineTo(triangle.c.x, triangle.c.y);
            path.lineTo(triangle.a.x, triangle.a.y);

            path.close();

            trianglePaint.setColor(colorGenerator.nextColor());
            canvas.drawPath(path, trianglePaint);
        }
```

## Locating the problem (2)
After I draw the path initialization out of onDraw(), I still can see a lot of "Point" objects in the memory, so I still have a long way to go.

And during the debuging, I found out that there are not so many objects in the memory at the first time. If I drag the seekbar, the number of objects expand rapidly. This give me a big inspiration.

I started to focus on the sample module, and found out the culprit:

```java
        cellSizeControl.setOnSeekBarChangeListener(new SeekBar.OnSeekBarChangeListener() {
            @Override
            public void onProgressChanged(SeekBar seekBar, int progress, boolean fromUser) {
                if (progress > 0) {
                    trianglifyView.setDrawingCacheEnabled(false);
                    trianglifyView.setCellSize(progress * 10);
                    trianglifyView.setDrawingCacheEnabled(true);
                }  
            }            

            @Override
            public void onStartTrackingTouch(SeekBar seekBar) {  }

            @Override
            public void onStopTrackingTouch(SeekBar seekBar)  {  }
```

If I drag the seekbar from 0 to 20, that means it will cause drawing action 20 times, and each of them will generate a lot of objects.  Further more, since the cellSize and variance is small, trianglify will generate a lot of points. (If you have a large cellSize, that means you only need few points. But If your cellSize is small, you have to generate way more Point to fulfill the small cell size.) And this is why I have so many objects in such a short time.

Since I figure out why the problems happens, now I can start to fix it. I can put the drawing action out of the SeekBarChangeListener. Only if I finished the dragging, I will start to draw a trianglified graphics

```java
        cellSizeControl.setOnSeekBarChangeListener(new SeekBar.OnSeekBarChangeListener() {
            int progress;
            @Override
            public void onProgressChanged(SeekBar seekBar, int progress, boolean fromUser) {
                this.progress = progress;
            }            

            @Override
            public void onStartTrackingTouch(SeekBar seekBar) { }

            @Override
            public void onStopTrackingTouch(SeekBar seekBar) {
                if (progress > 0) {
                    trianglifyView.setDrawingCacheEnabled(false);
                    trianglifyView.setCellSize(progress * 10);
                    trianglifyView.setDrawingCacheEnabled(true);
                }                
            }
```

This is much better, and ANR is off our sight. 