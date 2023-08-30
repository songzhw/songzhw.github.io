

# Issues of `rv.smoothScrollToPosition(pos)`
1). Let's say your RecyclerView(rv) shows the content of item 0 to item 3. Now if you want to smooth scroll(SS) to item 5,  then `rv.smoothScrollToPosition(5)` will only make the item 5 fully visible at the bottom of rv.

2). Let's say your RecyclerView(rv) shows the content of item 7 to item 10. Now if you want to smooth scroll(SS) to item 5,  then `rv.smoothScrollToPosition(5)` will only make the item 5 fully visible at the top of rv.

3). Let's say your RecyclerView(rv) shows the content of item 0 to item 3. Now if you want to smooth scroll(SS) to item 2,  then `rv.smoothScrollToPosition(2)` will do nothing. Literally nothing, no scroll at all. 

Most of time, PO want to the target item to show at the top of the rv, or at the center of rv. Not something inconsistant position (sometimes top, sometimes bottom). 

# Use case 1: smooth scroll and make the target item at the top
There is one possible solution:
```kotlin
val first = rv.layoutMananger.firstVisibleItemPosition()
val last = rv.layoutManager.lastVisibleItemPosition()
if(targetPosition < first) {
  rv.smoothScrollToPosition(targetPosition)
} 
else if(targetPosition < last) { // between [first, last]
  rv.smoothScrollBy(0, targetItem.top)
}
else {
  // this case might be complex. You have to listen to the scroll and scroll to target when the target item is found.  
  rv.smoothScrollTo(targetPosition)
  rv.addOnScrollListener{ 
    if(state == SCROLL_STATE_IDLE) {
      rv.scrollBy(0, targetitem.top)
    }
    rv.removeScrollListener(this)
  }
}
```

The solution is able to make the SS be more smart, but it's kind of complex. 
A more simpler solution would be use LiearSmoothScroll(LSS): 

```kotlin
    private fun RecyclerView.smoothScrollAndSnapStartTo(pos: Int) {
        val scroller = object : LinearSmoothScroller(context) {
            override fun getVerticalSnapPreference() = LinearSmoothScroller.SNAP_TO_START
        }
        scroller.targetPosition = pos
        layoutManager?.startSmoothScroll(scroller)
    }

```

# Use case 2: snap_to_start and add some offset

Now I have a row of buttons at the top of RV, and the `smoothScrollAndSnapStartTo(pos)` is not working very well.
![](./_image/img_20230829_001.png)
Because the row of buttons is blocking some of the target position, so UX don't think it's good enough to watch. So I have to make the SS to snap_to_start, and also add some offset to the SS.

## how SS works?
when you want to SS to the target position, the RV will start scrolling, until the target position shows. 
Then the scrolling enters the phase 2, which is the `Deceleration` phase. In this deceleration phase, LSS will try to calculate how long it will scroll and how much time it takes. The source code of LSS will tell us more.  It's `onTargetFound()` method is got called when the target positions start showing in the screen.

```java
// LSS - source code
@override
protected void onTargetFound(View targetView, RecyclerView.State state, Action action) {
    final int dx = calculateDxToMakeVisible(targetView, getHorizontalSnapPreference());
    final int dy = calculateDyToMakeVisible(targetView, getVerticalSnapPreference());
    final int distance = (int) Math.sqrt(dx * dx + dy * dy);
    final int time = calculateTimeForDeceleration(distance);
    if (time > 0) {
        action.update(-dx, -dy, time, mDecelerateInterpolator);
    }
}
```

And what is the `calculateDxToMakeVisible` and `calculateDyToMakeVisible`. These two methods are calculating how much distance the rv should scroll in the x direction and y direction. Their source code is something like this: 

```java
// LSS - source code

    public int calculateDxToMakeVisible(View view, int snapPreference) {
        ...
        return calculateDtToFit(viewLeft, viewRight, rvLeft, rvRight, snapPreference);
    }
    
    public int calculateDyToMakeVisible(View view, int snapPreference) {
        ...
        return calculateDtToFit(viewTop, viewBottom, rvTop, rvBottom, snapPreference);
    }    

```

Yes, they are both using the method `calculateDtToFit()`. And when we want to adjust the scroll distance, this `calculateDtToFit()` method is where it can help us.

## solution

```kotlin
   private fun RecyclerView.smoothScrollWithOffsetTo(pos: Int) {
        val topButtonsHeight = topButtonsLayout.height //=> 126

        val scroller = object : LinearSmoothScroller(context) {

            override fun getVerticalSnapPreference() = LinearSmoothScroller.SNAP_TO_START
            
            override fun calculateDtToFit(viewStart: Int, viewEnd: Int, boxStart: Int, boxEnd: Int, snapPreference: Int): Int {
                val computed = super.calculateDtToFit(viewStart, viewEnd, boxStart, boxEnd, snapPreference)
                if (snapPreference == SNAP_TO_START) {
                    return computed + topButtonsHeight
                }
                return computed
            }
        }
        scroller.targetPosition = pos
        layoutManager?.startSmoothScroll(scroller)
    }

```

Here is the result after I call `rv.smoothScrollWithOffsetTo(7)`. Now you can see the item 7 is not blocked by the top buttons row.
![](./_image/img_20230829_002.png)