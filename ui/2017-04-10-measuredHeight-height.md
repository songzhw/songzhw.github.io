
This week I read a post about how to make a ScrollView with the rebound effect. The article is actually good, but the author has a huge error.  I think I have to clarify, so that people will not get misguided again.

### Background
ScrollView with the rebound effect only show that effect when you already scroll the the bottom or the top. So we need to know when this is the time. Here is the author did :

```java
    private boolean isNeedMove() {
        int offset = innerView.getMeasuredHeight() - getHeight();
        int scrollY = getScrollY();
        if (scrollY == 0 || scrollY == offset) {
            return true;
        }
        return false;
    }
```


`scrollY == 0` means ScrollView is now scrolled to its top
`scrollY == offset` means ScrollView is now scrolled to its bottom

So the author has a conclusion:
When the height of one View is so huge that exceed the screen, then the **getMeasuredHeight()** is view's real height, and the **getHeight** is the screen height.  In that case, **getMeasuredHeight = getHeight() + height out of screen**.

### Why it is wrong?
The above function is right, but the conclusion is not that right.

