

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