---
title: Behavior
date: 2019-12-11 21:11:20
tags: 
    - Android 
---
# Behavior

## 方法分析
### 默认构造和用于xml的构造（从布局定义时）

~~~

       public Behavior() {
       }


       public Behavior(Context context, AttributeSet attrs) {
       }
~~~
### CoordinateLayout的LayoutParameter会保存Behavior引用，Behavior绑定到LayoutParameter和解绑时调用下面两个方法
~~~
       public void onDetachedFromLayoutParams() {
       }
~~~

### 是否拦截touchEvent
> * 在CoordinateLayout的点击事件分配到子View之前对其进行响应,
  Behavior可以用这个方法监听范围内的TouchEvent直到其中一个View决定拦截剩余的事件流（event stream）来对关联的子view做出相应动作。
  返回true拦截，返回false不拦截。
> * 一旦一个Behavior拦截了一个事件，剩余的事件流会被发送到Behavior的onTouchEvent方法
> * 无论关联的View是否可见，本方法都会被调用，如果想要在view可见时进行操作，对child判断isShown就可以了
> * 默认实现返回的一直是false
 #### 参数说明
* parent 接收事件的(CoordinateLayout)
* child 与Behavior关联的View
~~~
        public boolean onInterceptTouchEvent(CoordinatorLayout parent, V child, MotionEvent ev) {
            return false;
        }
~~~

### 点击事件接收处
>
* onInterceptTouchEvent方法返回true开始被调用,
* Behavior可以拦截触摸事件，以帮助CoordinatorLayout布局操纵其子视图。
例如，Behavior可能允许用户拖动一个UI窗格打开或关闭。 这个方法应该执行视图布局状态的实际变化
* 无论关联的View是否可见，本方法都会被调用，如果想要在view可见时进行操作，对child判断isShown就可以了
* 参数说明同上
~~~
        public boolean onTouchEvent(CoordinatorLayout parent, V child, MotionEvent ev) {
            return false;
        }
~~~

###  决定依赖的View
>
* 这个方法决定是否提供的子视图是另一个特定兄弟view的布局依赖
* 每次layout该方法都会被至少调用一次,如果返回true.那么子view和dependency就是一对依赖的关系,父布局(CoordinatorLayout)总是会在依赖的view布局完成后再对child进行layout,无论子view在其中的顺序如何,当依赖的view布局或者位置发生改变时还会调用onDependentViewChanged方法.
#### 参数说明
* parent 就是CoordinatorLayout, child就是需要检测的view,dependency是child建议的依赖

~~~
       public boolean layoutDependsOn(CoordinatorLayout parent, V child, View dependency) {
           return false;
       }
~~~

### layoutDependsOn方法
> * 设置的dependency view的位置或者布局发生变化时会调用.如果Behavior改变了子视图的大小或位置，它应该返回true。

~~~
/**
         * Respond to a change in a child's dependent view
         *
         * <p>This method is called whenever a dependent view changes in size or position outside
         * of the standard layout flow. A Behavior may use this method to appropriately update
         * the child view in response.</p>
         *
         * <p>A view's dependency is determined by
         * {@link #layoutDependsOn(CoordinatorLayout, android.view.View, android.view.View)} or
         * if {@code child} has set another view as it's anchor.</p>
         *
         * <p>Note that if a Behavior changes the layout of a child via this method, it should
         * also be able to reconstruct the correct position in
         * {@link #onLayoutChild(CoordinatorLayout, android.view.View, int) onLayoutChild}.
         * <code>onDependentViewChanged</code> will not be called during normal layout since
         * the layout of each child view will always happen in dependency order.</p>
         *
         * <p>If the Behavior changes the child view's size or position, it should return true.
         * The default implementation returns false.</p>
         *
         * @param parent the parent view of the given child
         * @param child the child view to manipulate
         * @param dependency the dependent view that changed
         * @return true if the Behavior changed the child view's size or position, false otherwise
         */
        public boolean onDependentViewChanged(CoordinatorLayout parent, V child, View dependency) {
            return false;
        }

~~~

### onMeasureChild方法
> * 测量子view
* @param parent the parent CoordinatorLayout
        * @param child the child to measure
        * @param parentWidthMeasureSpec the width requirements for this view
        * @param widthUsed extra space that has been used up by the parent
        *        horizontally (possibly by other children of the parent)
        * @param parentHeightMeasureSpec the height requirements for this view
        * @param heightUsed extra space that has been used up by the parent
        *        vertically (possibly by other children of the parent)
        * @return true if the Behavior measured the child view, false if the CoordinatorLayout
        *         should perform its default measurement


### nested-Scroll开始
> * 与CoordinatorLayout的任何直接子view相关的任何行为都可以作出回应,并返回true以表明CoordinatorLayout应该作为此滚动的嵌套滚动父级。只有返回true的行为此方法将接收后续的嵌套滚动事件。

~~~
/**
        * Called when a descendant of the CoordinatorLayout attempts to initiate a nested scroll.
        *
        * <p>Any Behavior associated with any direct child of the CoordinatorLayout may respond
        * to this event and return true to indicate that the CoordinatorLayout should act as
        * a nested scrolling parent for this scroll. Only Behaviors that return true from
        * this method will receive subsequent nested scroll events.</p>
        *
        * @param coordinatorLayout the CoordinatorLayout parent of the view this Behavior is
        *                          associated with
        * @param child the child view of the CoordinatorLayout this Behavior is associated with
        * @param directTargetChild the child view of the CoordinatorLayout that either is or
        *                          contains the target of the nested scroll operation
        * @param target the descendant view of the CoordinatorLayout initiating the nested scroll
        * @param axes the axes that this nested scroll applies to. See
        *                         {@link ViewCompat#SCROLL_AXIS_HORIZONTAL},
        *                         {@link ViewCompat#SCROLL_AXIS_VERTICAL}
        * @param type the type of input which cause this scroll event
        * @return true if the Behavior wishes to accept this nested scroll
        *
        * @see NestedScrollingParent2#onStartNestedScroll(View, View, int, int)
        */
       public boolean onStartNestedScroll(@NonNull CoordinatorLayout coordinatorLayout,
               @NonNull V child, @NonNull View directTargetChild, @NonNull View target,
               @ScrollAxis int axes, @NestedScrollType int type) {
           if (type == ViewCompat.TYPE_TOUCH) {
               return onStartNestedScroll(coordinatorLayout, child, directTargetChild,
                       target, axes);
           }
           return false;
       }
~~~

### 当嵌套滚动结束时调用
~~~
/**
         * Called when a nested scroll has ended.
         *
         * <p>Any Behavior associated with any direct child of the CoordinatorLayout may elect
         * to accept the nested scroll as part of {@link #onStartNestedScroll}. Each Behavior
         * that returned true will receive subsequent nested scroll events for that nested scroll.
         * </p>
         *
         * <p><code>onStopNestedScroll</code> marks the end of a single nested scroll event
         * sequence. This is a good place to clean up any state related to the nested scroll.
         * </p>
         *
         * @param coordinatorLayout the CoordinatorLayout parent of the view this Behavior is
         *                          associated with
         * @param child the child view of the CoordinatorLayout this Behavior is associated with
         * @param target the descendant view of the CoordinatorLayout that initiated
         *               the nested scroll
         * @param type the type of input which cause this scroll event
         *
         * @see NestedScrollingParent2#onStopNestedScroll(View, int)
         */
        public void onStopNestedScroll(@NonNull CoordinatorLayout coordinatorLayout,
                @NonNull V child, @NonNull View target, @NestedScrollType int type) {
            if (type == ViewCompat.TYPE_TOUCH) {
                onStopNestedScroll(coordinatorLayout, child, target);
            }
        }
~~~

### 当正在进行嵌套滚动更新并且目标已经滚动或试图滚动

~~~
/**
         * Called when a nested scroll in progress has updated and the target has scrolled or
         * attempted to scroll.
         *
         * <p>Any Behavior associated with the direct child of the CoordinatorLayout may elect
         * to accept the nested scroll as part of {@link #onStartNestedScroll}. Each Behavior
         * that returned true will receive subsequent nested scroll events for that nested scroll.
         * </p>
         *
         * <p><code>onNestedScroll</code> is called each time the nested scroll is updated by the
         * nested scrolling child, with both consumed and unconsumed components of the scroll
         * supplied in pixels. <em>Each Behavior responding to the nested scroll will receive the
         * same values.</em>
         * </p>
         *
         * @param coordinatorLayout the CoordinatorLayout parent of the view this Behavior is
         *                          associated with
         * @param child the child view of the CoordinatorLayout this Behavior is associated with
         * @param target the descendant view of the CoordinatorLayout performing the nested scroll
         * @param dxConsumed horizontal pixels consumed by the target's own scrolling operation
         * @param dyConsumed vertical pixels consumed by the target's own scrolling operation
         * @param dxUnconsumed horizontal pixels not consumed by the target's own scrolling
         *                     operation, but requested by the user
         * @param dyUnconsumed vertical pixels not consumed by the target's own scrolling operation,
         *                     but requested by the user
         * @param type the type of input which cause this scroll event
         *
         * @see NestedScrollingParent2#onNestedScroll(View, int, int, int, int, int)
         */
        public void onNestedScroll(@NonNull CoordinatorLayout coordinatorLayout, @NonNull V child,
                @NonNull View target, int dxConsumed, int dyConsumed,
                int dxUnconsumed, int dyUnconsumed, @NestedScrollType int type) {
            if (type == ViewCompat.TYPE_TOUCH) {
                onNestedScroll(coordinatorLayout, child, target, dxConsumed, dyConsumed,
                        dxUnconsumed, dyUnconsumed);
            }
        }
~~~

### 当正在进行嵌套滚动即将更新时，并且早在目标消耗任何滚动距离之前 ,本方法将被调用。
~~~
/**
         * Called when a nested scroll in progress is about to update, before the target has
         * consumed any of the scrolled distance.
         *
         * <p>Any Behavior associated with the direct child of the CoordinatorLayout may elect
         * to accept the nested scroll as part of {@link #onStartNestedScroll}. Each Behavior
         * that returned true will receive subsequent nested scroll events for that nested scroll.
         * </p>
         *
         * <p><code>onNestedPreScroll</code> is called each time the nested scroll is updated
         * by the nested scrolling child, before the nested scrolling child has consumed the scroll
         * distance itself. <em>Each Behavior responding to the nested scroll will receive the
         * same values.</em> The CoordinatorLayout will report as consumed the maximum number
         * of pixels in either direction that any Behavior responding to the nested scroll reported
         * as consumed.</p>
         *
         * @param coordinatorLayout the CoordinatorLayout parent of the view this Behavior is
         *                          associated with
         * @param child the child view of the CoordinatorLayout this Behavior is associated with
         * @param target the descendant view of the CoordinatorLayout performing the nested scroll
         * @param dx the raw horizontal number of pixels that the user attempted to scroll
         * @param dy the raw vertical number of pixels that the user attempted to scroll
         * @param consumed out parameter. consumed[0] should be set to the distance of dx that
         *                 was consumed, consumed[1] should be set to the distance of dy that
         *                 was consumed
         * @param type the type of input which cause this scroll event
         *
         * @see NestedScrollingParent2#onNestedPreScroll(View, int, int, int[], int)
         */
        public void onNestedPreScroll(@NonNull CoordinatorLayout coordinatorLayout,
                @NonNull V child, @NonNull View target, int dx, int dy, @NonNull int[] consumed,
                @NestedScrollType int type) {
            if (type == ViewCompat.TYPE_TOUCH) {
                onNestedPreScroll(coordinatorLayout, child, target, dx, dy, consumed);
            }
        }
~~~
