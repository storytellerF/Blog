# [Android]通过setOnTouchListener实现移动View功能

## 需求

有时需要移动View，以前写的代码不太好，就上网上找，也不好用，和我自己写的都有的问题是View 会比光标的位置靠右下方。以前都忍了，偏就偏吧，现在决心要改，也可能是因为在这个特殊时期比较闲吧。

## 实现

正如上面所说，靠右下方，而且相比于靠右，靠下的程度更大。这很难不让人想到是状态栏造成的影响，而且单单是状态栏都还不够，还有一个`ToolBar`，所以要点在于如何去除这个高度。

```java

    /**
     * Returns the original raw Y coordinate of this event.  For touch
     * events on the screen, this is the original location of the event
     * on the screen, before it had been adjusted for the containing   window
     * and views.
     *
     * @see #getY(int)
     * @see #AXIS_Y
     */
    public final float getRawY() {
        return nativeGetRawAxisValue(mNativePtr, AXIS_Y, 0, HISTORY_CURRENT);
    }
```

`getRawX` 或者`getRawY` 返回的都是相对于整个屏幕的，在测试时，如果不加限制，甚至可以滑动到状态栏，事件一直有效。

```java
    /**
     * {@link #getY(int)} for the first pointer index (may be an
     * arbitrary pointer identifier).
     *
     * @see #AXIS_Y
     */
    public final float getY() {
        return nativeGetAxisValue(mNativePtr, AXIS_Y, 0, HISTORY_CURRENT);
    }
```

可是看这个`getX` 函数根本不明白说的什么意思，甚至它们的实现完全一样，可能是`mNativePtr` 或者`HISTORY_CURRENT` 有什么不同吧，不管了。到网上查，是相对这个我们设置触摸事件的`View` 的位置。

在每次设定位置时，需要的是设置当前的`View` 相对于父`View` 的坐标，在每次移动时`getRawY` 的值需要减去`getY` 的值，得到的值就是当前view相对整个屏幕的位置，在减去当前`view` 相对于父窗体的位置，这样得到的值就是父窗体相对于整个屏幕的位置，最后得到的值是跟我们没有关系的，在获得当前`view` 需要的位置时在减去这个值。父窗体是不会动的，所以后者的这个位置可以在`MotionEvent.ACTION_DOWN` 时获取，同样的，`getY` 的值也需要在这个时候获取，因为光标相对于当前的`View` 的位置也是不能改变的。
像这样：

```java
case MotionEvent.ACTION_DOWN:
    x = event.getX();
    y = event.getY();
    left = event.getRawX() - view.getLeft() - x;
    top = event.getRawY() - view.getTop() - y;
    return true;
```

当用户只是点击时，是不会有`MotionEvent.ACTION_MOVE` 事件的，所以在这个事件下记录当前用户是否进行的是移动操作，而不是在上面的。

```java
case MotionEvent.ACTION_MOVE:
    if (!moved) {
        moved = true;
    }
    setXPosition(event.getRawX() - x - left);
    setYPosition(event.getRawY() - y - top);
    return true;
```

我们用光标相对于整个屏幕的位置减去父窗体的位置，减去光标相对于父窗体的位置，就是当前`view` 的左上角相对于父窗体的位置。

当用户抬手时，判断是否是移动了，如果移动了，那就不是点击事件，而且这个移动操作也结束了，恢复原样，如果不是，那就是点击事件，就调用`performClick` 函数。

```java
case MotionEvent.ACTION_UP:
    if (moved) {
        moved = false;
    } else {
        v.performClick();
    }
    return true;
```

至于那个`performClick`：

```java
    /**
     * Call this view's OnClickListener, if it is defined.  Performs all normal
     * actions associated with clicking: reporting accessibility event, playing
     * a sound, etc.
     *
     * @return True there was an assigned OnClickListener that was called, false
     *         otherwise is returned.
     */

```

这样`onClickListener` 就能够正常运行了，而不是被我们阻止了。

这还没有完：

```java
    /**
     * Entry point for {@link #performClick()} - other methods on View should call it instead of
     * {@code performClick()} directly to make sure the autofill manager is notified when
     * necessary (as subclasses could extend {@code performClick()} without calling the parent's
     * method).
     */
    private boolean performClickInternal() {
        // Must notify autofill manager before performing the click actions to avoid scenarios where
        // the app has a click listener that changes the state of views the autofill service might
        // be interested on.
        notifyAutofillManagerOnClick();

        return performClick();
    }
```

他提醒我不让我直接调用`performClick` ，而是使用`performClickInternal` ，说是为了保证`autofill manager` 工作。可是这个函数是个私有函数，可能说的不是我的这种情况吧。

至于`perfomClick` 的返回值，感觉不重要就不管了。

## 最后

如果是悬浮窗，`view.getTop()` 返回的总是0，所以需要通过`LayoutParams`来获取这个`top` 的值,并且left不再需要。

```java

    WindowManager.LayoutParams layoutParams= (WindowManager.LayoutParams) v.getLayoutParams();
    top = event.getRawY() -layoutParams.y- y;

```
