Last time when I went to an interview, I was asked to make a expandlable view, just like the ExpandableListView. I wrote down what I thought in the white board, which turns out true. But I am still interested in making a real one. So here it it.

## 1. What it looks like?

![](./_image/expandable_layout.gif)

Every time you click the title, the layout will toggle to show or hide the content view

At our first glance, what might be the difficulties we have?
* How much height does the content view has?
* How to hide the content view at first, and then get its real height? 
* How to do the Animation?

## 2. get children view
It would be a good idea to just draw the static layout first when you try to draw a view and its animation. This way you break down the big issue into small issues.

Based on the layout of the title view and the content view, we obviously think that the layout could be a vertical LinearLayout. 

```java
public class ExpandableLayout extends LinearLayout {
        public void init() {  // called by constructors
            this.setOrientation(VERTICAL);
        }
}
```

Apparently every pages using ExpandableLayout may have different title view and content view. So the two children views should be customizable. Here, we leave it to the user. Users can add the children views they want in their layout xmls.

Hence, we require two, and only two, children views within this ExpandableLayout. Because `onFinishInflater()` will get called after this View has been inflated to the screen, here is the code to get the children view. 

```java
    @Override
    protected void onFinishInflate() {
        super.onFinishInflate();
        if (getChildCount() != 2) {
            throw new RuntimeException("ExpandableLayout could only have two children: header and content!");
        }        
        headerView = getChildAt(0);
        contentView = getChildAt(1);

        headerView.setOnClickListener(v -> {
            toggle();
        });

    }
```

## 3. get child height
We know that `onSizeChange()` occurs after `onMeasure()`, so we could get the children size in `onSizeChanged()`. Here is what I did.

```java
    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        contentHeight = contentView.getMeasuredHeight(); 
        contentView.setVisibility(View.GONE); // to make the content view disappear when we first launch this page
    }
```


## 4. toggle the layout
When we reach here, we already know the content view, and its height. Now we need to make it toggle when we click the header view. 

Toggling is not hard. It's all about an animator of height change, and its reversed animation.

```java
    public void toggle(){
        if(isExpanded){
            collapse();
        } else {
            expand();
        }
        isExpanded = !isExpanded;
    }

    public void collapse(){
    	// TODO 
    }

    public void expand(){
    	// TODO
    }
```

To make the animatin we need, we need to change the content view's height. So we need to change its LayoutManager.height. 

```java
    public void collapse(){
        ViewGroup.LayoutParams lp = contentView.getLayoutParams();
        ValueAnimator animator = ObjectAnimator.ofInt(contentHeight, 0);
        animator.addUpdateListener( anim -> {
            lp.height = (int) anim.getAnimatedValue();
            contentView.setLayoutParams(lp);
        });
        animator.start();
    }

    public void expand(){
        contentView.setVisibility(View.VISIBLE);

        ViewGroup.LayoutParams lp = contentView.getLayoutParams();
        ValueAnimator animator = ObjectAnimator.ofInt(0, contentHeight);
        animator.addUpdateListener( anim -> {
            lp.height = (int) anim.getAnimatedValue();
            contentView.setLayoutParams(lp);
        });
        animator.start();
    }
```

## 5. No animation? 
Clicking the title, we see no animation. Odd. Let's find out the root cause. I placed some log in the code, and find out the contentHeight is 0 again.

Oh, now I get it. We've set the visibility to GONE, so the `onSizeChanged()` get called again,  therefore the content view's height is 0 again.

So I manage to make sure that the getting child height logic is called only once again when its initial state is collapsed.

```java
[existing code]
    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        contentHeight = contentView.getMeasuredHeight(); 
        contentView.setVisibility(View.GONE); // to make the content view disappear when we first launch this page
    }
```

Here we deleted the `contentView.setVisibility(View.GONE);`, and replace it with the change of LayoutParams.

```java
[new code]
private int contentHeight = -1;
    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        if (contentHeight == -1) {
            contentHeight = contentView.getMeasuredHeight();
        }
        contentView.setVisibility(View.GONE); 
    }
```

## 6. Collapse it 
Now the clicking event works fine. You can expand or collapse the content when you click the title. But the code above make the initial state of our ExpandableLayout to be expanded. This obviously means the `contentView.setVisibility(View.GONE); ` in the `onSizeChanged()` does not work. I need to fix that too.

Then I may have another solution. `onSizeChanged()` is not the only place we could get the view height. In fact, `onMeasure()` is also qualified for this job. I was thinking using `onMeasured()` to initialize the collapsed state.

Here is the change I made. 
```java
    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);

        if (contentHeight == -1) {
            contentHeight = contentView.getMeasuredHeight(); //此时contentView.getHeight()仍是0
        }

        lp = contentView.getLayoutParams();
        lp.height = 0;
        contentView.setLayoutParams(lp);
    }

    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
    }
```

