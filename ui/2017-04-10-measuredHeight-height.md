
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
The above function is right, but the conclusion is not that right. Normally, getMeasuredHeight() equals getHeight(), even the view is very huge that already exceed the screen!

Looking at the Android source code, we know, *measuredHeight* is determined by the View.onMeasure(), and *height* is determined by the View.onLayout().  If you are customing a View, it's not a good idea to use *getHeight()* to get the View's height in its onLayout() method, because by then the *height* has not been calculated yet. Instead, you should use *getMeasuedHeight()*. And in other cases, measuredHeight equals to height. 

### Confused?
If the author is wrong, then why `scrollY == offset` can tell us that we are already scrolling to the end?

That's a little tricky, you have to pay more attention to it. Now look at that code again, do you found something wrong?
: Yes, the code is `int offset = innerView.getMeasuredHeight() - getHeight();`, 
and it is not `int offset = getMeasuredHeight() - getHeight();`!

Say we have a ScrollView that is fill the whole screen (e.g. height = 1920), and its child is even huger, height is 3020. `innerView` is the only child view of our ScrollView. And here is the numbers:
```
innerView.measuredHeight = 3020  ; innerView.height = 3020 ; 
scrollView.measuredHeight = 1920 ; scrollView.height = 1920 ; 
```

If you are trying scroll to the bottom, the scrollY will get bigger and bigger. When you reached to the bottom, the scrollY will be
`scrollY = 1100`

Now we are understand why the author made such a mistake. He/She thought the code is `int offset = getMeasuredHeight() - getHeight();`. But they are not. In fact, in this case, the offset will always be 0.

### MeasureSpec.UNSPECIFIED

The source code of ScollView has a method called "measureChild()". In this "measureChild" method, we saw such a line:

```java
childHeightMeasureSpec = MeasureSpec.makeSafeMeasureSpec(
        Math.max(0, MeasureSpec.getSize(parentHeightMeasureSpec) - verticalPadding),
        MeasureSpec.UNSPECIFIED);
```

The MeasureSpec mode is UNSPECIFIED. So what is UNSPECIFIED?
:  The parent has not imposed any constraint on the child. It can be whatever size it wants.

Oh, then we can get it. That's why the LinearLayout in the above example has the 3000px height, and the scrollView only have 1136px height.  Because the LinearLayout is UNSPECIFIED, so it can be whatever it wants, that's why the height is bigger than scrollView's height.















