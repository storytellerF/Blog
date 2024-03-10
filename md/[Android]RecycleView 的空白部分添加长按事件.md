# [Android]RecycleView 的空白部分添加长按事件

## 需求

需要一个类似电脑上的那种，即使是空白也能够长按，虽然空白部分与非空白部分不同。

## 探求

1. 查找了一些资料，有一个讲到一个奇怪的东西，类型与自动识别了所有的操作类型，其中有长按，只可惜找不到了，在Android 中也查不到，我恨我自己当时没有测试。

2. 然后去查看View 的长按事件，感觉好麻烦，好像监听事件不是由View 本身去做的。

3. 然后在`stackoverflow` 上找到了，办法是通过`OnItemTouchListener`。

## 解决

1. 方法是

    ```java
       /**
        * Add an {@link OnItemTouchListener} to intercept touch events before they are dispatched
        * to child views or this view's standard scrolling behavior.
        *
        * <p>Client code may use listeners to implement item manipulation behavior. Once a listener
        * returns true from
        * {@link OnItemTouchListener#onInterceptTouchEvent(RecyclerView, MotionEvent)} its
        * {@link OnItemTouchListener#onTouchEvent(RecyclerView, MotionEvent)} method will be called
        * for each incoming MotionEvent until the end of the gesture.</p>
        *
        * @param listener Listener to add
        * @see SimpleOnItemTouchListener
        */
        public void addOnItemTouchListener(@NonNull OnItemTouchListener listener) {
            mOnItemTouchListeners.add(listener);
        }
    ```

    说的是会在分发到子对象和滚动事件之前拦截。
    如果`onInterceptTouchEvent` 返回值`true` 后面一连串的事件都由`onTouchEvent` 处理，不过我们当前用不到，直接返回`false` 得了。

2. 添加，并实现`onInterceptTouchEvent`。

    ```java
        Log.d(TAG, "onInterceptTouchEvent() called with: rv = [" + rv + "], e = [" + e + "]");
        if (e.getAction() == MotionEvent.ACTION_UP) {
            long l = e.getEventTime() - e.getDownTime();
            Log.i(TAG, "onTouchEvent: l:"+l);
            if (l > 900) {
                //TODO long click event
            }
            }
        }
        return false;
    ```

    事件本事会记录按下时间，当取消按下时获取时间差即可。
    实际测试，感觉设置`900` 即可。
    如果需要长按中滑动就取消此次事件的话请自行实现。
