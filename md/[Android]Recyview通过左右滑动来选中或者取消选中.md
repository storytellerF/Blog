# [Android]RecyView通过ItemTouchHelper实现左右滑动来选中或者取消选中的功能

## 需求

需要为RecyView 添加一个多选的功能，这里我想通过ItemTouchHelper 实现

## 步骤

1. 新建一个类，继承自`ItemTouchHelper.Callback` (关于`ItemTouchHelper` 如何使用这里不讲）。完成之后，发现当你滑动时，这个`Viewholder` 真的不见了，这不是我想要的，我希望它能够滑动，但是不能够被移除。
开始解决问题。

2. 重写`getSwipeThreshold` 方法。

      ```java

      /**
      * Returns the fraction that the user should move the View to be considered as swiped.
      * The fraction is calculated with respect to RecyclerView's bounds.
      * <p>
      * Default value is .5f, which means, to swipe a View, user must move the View at least
      * half of RecyclerView's width or height, depending on the swipe direction.
      *
      * @param viewHolder The ViewHolder that is being dragged.
      * @return A float value that denotes the fraction of the View size. Default value
      * is .5f .
      */
      @SuppressWarnings("WeakerAccess")
      public float getSwipeThreshold(@NonNull ViewHolder viewHolder) {
            return .5f;
      }
      ```

      说的是当用户移动多少时认为是用户进行了移除行为。这个`fraction` 是的小数的意思，也就是说相当于`RecycleView` 的宽高多少长度。这是我想要的东西。

      ```java

      @Override
      public float getSwipeThreshold(@NonNull RecyclerView.ViewHolder viewHolder) {
            return 10;
      }
      ```

      像这样返回`1000%`,就成了。

3. 但是，又出现问题了，移动过快也会导致移除事件。需要`getSwipeEscapeVelocity` 。

      ```java

      /**
      * Defines the minimum velocity which will be considered as a swipe action by the user.
      * <p>
      * You can increase this value to make it harder to swipe or decrease it to make it easier.
      * Keep in mind that ItemTouchHelper also checks the perpendicular velocity and makes sure
      * current direction velocity is larger then the perpendicular one. Otherwise, user's
      * movement is ambiguous. You can change the threshold by overriding
      * {@link #getSwipeVelocityThreshold(float)}.
      * <p>
      * The velocity is calculated in pixels per second.
      * <p>
      * The default framework value is passed as a parameter so that you can modify it with a
      * multiplier.
      *
      * @param defaultValue The default value (in pixels per second) used by the
      *                     ItemTouchHelper.
      * @return The minimum swipe velocity. The default implementation returns the
      * <code>defaultValue</code> parameter.
      * @see #getSwipeVelocityThreshold(float)
      * @see #getSwipeThreshold(ViewHolder)
      */
      @SuppressWarnings("WeakerAccess")
      public float getSwipeEscapeVelocity(float defaultValue) {
            return defaultValue;
      }
      ```

      说的是定义一个最小速度用来检测当前的滑动是否当成移除操作。然后我们设置一个很大的值就行了。注释里还说了`getSwipeVelocityThreshold` 这个函数，理解起来应该是说怕我`SwipeEscapeVelocity` 设置的过小，导致错把上下滑动当成了左右滑动吧，与我无关，不管了。

      ```java

      @Override
      public float getSwipeEscapeVelocity(float defaultValue) {
            return 1000000;
      }
      ```

4. 这样用户不管怎么滑都不会移除了。但是不能移除，`onSwiped` 就不会被调用了，这该如何是好？可以重写`clearView` 函数，当移除取消时，这个函数就会被调用。之前也想过在前面直接给它拦截了，但是不行，全部没有权限，只有这个是可以使用的。
      通过`getSwipeThreshold` 可以不断的找到调用者为`checkHorizontalSwipe` ，`swipeIfNecessary` ，`select`，找到`select` 就够了，
      部分代码：

      ```java

      super.onAnimationEnd(animation);
      if (this.mOverridden) {
            return;
      }
      if (swipeDir <= 0) {
            // this is a drag or failed swipe. recover immediately
            mCallback.clearView(mRecyclerView, prevSelected);
            // full cleanup will happen on onDrawOver
      } else {
            // wait until remove animation is complete.
            mPendingCleanup.add(prevSelected.itemView);
            mIsPendingCleanup = true;
            if (swipeDir > 0) {
                  // Animation might be ended by other animators during a layout.
                  // We defer callback to avoid editing adapter during a layout.
                  postDispatchSwipe(this, swipeDir);
            }
      }
      ```

      我们要的就是
      >this is a drag or failed swipe. recover immediately

      这段话的意思是在执行拖拽或者滑动失败时，用来恢复的。所以，如果你还有拖拽的功能就比较麻烦了。

5. 最后全部完成了，但是，如果能够限制用户左右滑动的长度就更好了，只可惜当前还没有找到办法。
      现在找到办法了，`onChildDraw`函数。

      ```java

      /**
      * Called by ItemTouchHelper on RecyclerView's onDraw callback.
      * <p>
      * If you would like to customize how your View's respond to user interactions, this is
      * a good place to override.
      * <p>
      * Default implementation translates the child by the given <code>dX</code>,
      * <code>dY</code>.
      * ItemTouchHelper also takes care of drawing the child after other children if it is being
      * dragged. This is done using child re-ordering mechanism. On platforms prior to L, this
      * is
      * achieved via {@link android.view.ViewGroup#getChildDrawingOrder(int, int)} and on L
      * and after, it changes View's elevation value to be greater than all other children.)
      *
      * @param c                 The canvas which RecyclerView is drawing its children
      * @param recyclerView      The RecyclerView to which ItemTouchHelper is attached to
      * @param viewHolder        The ViewHolder which is being interacted by the User or it was
      *                          interacted and simply animating to its original position
      * @param dX                The amount of horizontal displacement caused by user's action
      * @param dY                The amount of vertical displacement caused by user's action
      * @param actionState       The type of interaction on the View. Is either {@link
      *                          #ACTION_STATE_DRAG} or {@link #ACTION_STATE_SWIPE}.
      * @param isCurrentlyActive True if this view is currently being controlled by the user or
      *                          false it is simply animating back to its original state.
      * @see #onChildDrawOver(Canvas, RecyclerView, ViewHolder, float, float, int,
      * boolean)
      */
      public void onChildDraw(@NonNull Canvas c, @NonNull RecyclerView recyclerView,
            @NonNull ViewHolder viewHolder,
            float dX, float dY, int actionState, boolean isCurrentlyActive) {
      ItemTouchUIUtilImpl.INSTANCE.onDraw(c, recyclerView, viewHolder.itemView, dX, dY,
                  actionState, isCurrentlyActive);
      }

      ```

      >If you would like to customize how your View's respond to user interactions, this isa good place to override.

      翻译过来就是

      >如果你想要自定义`View` 响应用户操作的，可以在这里重写。

      在使用时，可以通过`actionState` 来筛选移除和移动操作。

      重写时，我们需要的是操作完全不变，只有能够移动的位置不同。方法有两种。

      1. 在`dX` 大于某一个值时，不在使用`super` 调用父类的方法，这样程序什么都不会做。

            ```java

            @Override
            public void onChildDraw(@NonNull Canvas c, @NonNull RecyclerView recyclerView, @NonNull RecyclerView.ViewHolder viewHolder, float dX, float dY, int actionState, boolean isCurrentlyActive) {
                  if (actionState == ItemTouchHelper.ACTION_STATE_SWIPE) {
                        if (dX < 100) {
                              super.onChildDraw(c, recyclerView, viewHolder, dX, dY, actionState, isCurrentlyActive);
                        }
                  }
            }

            ```

      2. 不管`dX` 值有多大，都除以某一个值，比如`5`，这样就好带有阻尼效果一样，虽然拉的很远，带来的影响却很小。还可以根据自己的需要，在`dX` 不同值时选择不同的阻尼。

            ```java

            @Override
            public void onChildDraw(@NonNull Canvas c, @NonNull RecyclerView recyclerView, @NonNull RecyclerView.ViewHolder viewHolder, float dX, float dY, int actionState, boolean isCurrentlyActive) {
                  if (actionState == ItemTouchHelper.ACTION_STATE_SWIPE) {
                        super.onChildDraw(c, recyclerView, viewHolder, dX / 5, dY, actionState, isCurrentlyActive);
                  }
            }

            ```

6. 更新
      上面说的还不够，后续使用时，发现有时候事件没有正常接收到。但是在解决上一个问题时发现了一个`onChildDraw` 函数，可以通过这个函数来处理滑动事件而不是`clearView`。

      ```java
      if (actionState == ItemTouchHelper.ACTION_STATE_SWIPE) {
            int first_line = 200;
            if (dX < first_line) {
                  if (swipeEvent) {//这里的`真(true)`是上次触发了事件,用户滑动了回来（在这里可能也可以设置一个取消事件，不过效果不是很好），或者用户第二次滑动，重置为false，为的是用户滑动超过300时能够触发事件
                        swipeEvent=false;
                  }
                  super.onChildDraw(c, recyclerView, viewHolder, dX / 2, dY, actionState, isCurrentlyActive);
            } else {
                  int second_line = first_line + 100;
                  if (dX < second_line) {
                        int first_max = first_line / 2;
                        super.onChildDraw(c, recyclerView, viewHolder, first_max + (dX - first_line) / 4, dY, actionState, isCurrentlyActive);
                  } else if (dX >= second_line) {
                        if (!swipeEvent) {//如果没有触发过事件，触发一次，并且重置为true，保证下次不会触发
                              swipeViewHolder(viewHolder);
                              swipeEvent=true;
                        }
                  }
            }

        }
      ```

      swipeViewHolder就是需要自己实现的事件调用部分。

      滑动的距离小于200时，绘制的长度减半，大于200，小于300时绘制的长度在原来的基础上缓慢增加，大于300时就认为触发了事件，并且保证了事件不会被多次触发。
