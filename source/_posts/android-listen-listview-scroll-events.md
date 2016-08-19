---
title: ListView滚动事件的监听
date: 2014-08-06 10:25:35
tags: [Android, listview, 监听, 滚动事件, 表头表尾]
categories: android
---

ListView本身有一个OnScrollListener的接口，通过此接口可以监听ListView的状态，滚动的个数，但是没法监听其精确的滚动距离、上下滚动事件和滚动到表头或表尾事件，下面介绍如何监听这些事件。

## 1. 上下滚动事件的响应

1. 自定义一个继承至ListView的类。
2. 覆写onTouchEvent方法，监听触摸事件。
3. 定义一个有效滚动距离，当滑动的距离超过这个距离时才会触发滚动事件的响应。

```java
private int mEffectiveDistance = 10;
private float mPreviousY;

@Override
public boolean onTouchEvent(MotionEvent ev) {
    mDetector.onTouchEvent(ev);
    switch (ev.getAction()) {
        case MotionEvent.ACTION_MOVE:
            float currentY = ev.getY();
            float distanceY = currentY - mPreviousY;
            if (distanceY < -mEffectiveDistance) {
                mPreviousY = currentY;
                if (mCallbacks != null) {
                    mCallbacks.onScrollUp();  //触发向上滚动的事件
                }
            } else if (distanceY > mEffectiveDistance) {
                mPreviousY = currentY;
                if (mCallbacks != null) {
                    mCallbacks.onScrollDown();  //触发向下滚动的事件
                }
            }
            break;

        case MotionEvent.ACTION_DOWN:
            mPreviousY = ev.getY();
            break;

        default:
            break;

    }
    return super.onTouchEvent(ev);
}
```
<!-- more -->

## 2. 滚动到表头或表尾事件的响应

1. 自定义的ListView类要实现AbsListView.OnScrollListener接口。
2. 覆写onScrollStateChanged(AbsListView view, int scrollState)方法，监听列表的滚动事件，
其中view就是当前ListView，scrollState表示当前ListView的状态，它有三个状态：
SCROLL_STATE_IDLE（停止滚动的空闲状态）、
SCROLL_STATE_TOUCH_SCROLL（手指有接触屏幕并且带动ListView滚动的状态）、
SCROLL_STATE_FLING（手指离开的屏幕，ListView因为惯性还在滚动的状态）。
3. 当当前ListView状态是SCROLL_STATE_IDLE时进行事件监听。

### 2.1 滚动到表头的事件
当列表中的第一个子View的顶部Y坐标等于列表顶部Y坐标时响应
```java
@Override
public void onScrollStateChanged(AbsListView view, int scrollState) {
    if (scrollState == OnScrollListener.SCROLL_STATE_IDLE) {
        View firstChild = view.getChildAt(0);
        if (view.getTop() == firstChild.getTop()) {
            if (mCallbacks != null) {
                mCallbacks.onScrollFirst();   //触发滚动到表头的事件
            }
        }
    }
}
```

### 2.2. 滚动到表尾的事件
- 当列表中的最后一个子View的底部Y坐标等于列表底部Y坐标时响应
- 如果整个列表的长度比父View小，那么当最后一个子View的底部Y坐标等于列表的长度时响应
```java
@Override
public void onScrollStateChanged(AbsListView view, int scrollState) {
    int count = getCount();
    if (scrollState == OnScrollListener.SCROLL_STATE_IDLE
            && getLastVisiblePosition() == count - 1) {
        View lastChild = view.getChildAt(view.getChildCount() - 1);
        int dist = view.getBottom() - lastChild.getBottom();
        if (dist == 0) {
            if (mCallbacks != null) {
                mCallbacks.onScrollLast();   //触发滚动到表尾的事件
            }
        }
        if (count < MIN_CHILD_COUNT) {   //设置一个最小个数，提高效率
            int childViewsHeight = (lastChild.getHeight()
                    + getDividerHeight()) * count;
            if (childViewsHeight < view.getHeight() && childViewsHeight == lastChild.getBottom()) {
                if (mCallbacks != null) {
                    mCallbacks.onScrollLast();   //触发滚动到表尾的事件
                }
            }
        }
    }
}
```
