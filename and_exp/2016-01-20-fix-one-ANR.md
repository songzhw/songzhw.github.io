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


**[to be continued]**

