This is the final work of this animation.

![](./_image/20230915001.gif)

This animation seems simple as it only change the shape of our ImageView. But Android does not provide a shape animation API for us to do so. This means we have to define this animation by ourselves.

# key point
The key point of this animation is to draw the rect and the circle out first, and how to transform from one to another. 

Here we use `canvas.clipPath(path)`, the `path` here is a circle. But when the view is rectangle, then the radius of the circle is `max(width, height)`, this way, we would see the whole rectangle.
When the view is circle, the radius changed to `min(width, height) / 2.0`. 

```kotlin
   var isCircle = false 

    override fun onSizeChanged(w: Int, h: Int, oldw: Int, oldh: Int) {
        super.onSizeChanged(w, h, oldw, oldh)
        radius = max(width, height).toFloat()
        resetPaths()
        initAnimation()
    }

    override fun onDraw(canvas: Canvas) {
        canvas.clipPath(clipPath)
        super.onDraw(canvas) // image view draw the pictures
    }

    // = = = = = = = = = = = private func = = = = = = = = = = =
    private fun resetPaths() {
        val cx = width / 2f
        val cy = height / 2f 
        clipPath.reset()
        clipPath.addCircle(cx, cy, radius, Path.Direction.CW)
    }

```

# make the animation