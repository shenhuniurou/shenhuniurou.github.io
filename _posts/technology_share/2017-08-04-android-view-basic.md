---
layout: post
title: View的基础知识
category: blog
tags: Android、View
---


## View的概念

View是Android中非常重要的，它算是Android学习体系中比较难啃的一块骨头了，但是用户与一个app的交互就是与View的交互，也就是说用户无时无刻不在与app中的View打交道，所以把View的知识掌握透彻是一个Android开发者必要任务。View所有不在，基本上一个app里面我们能看见的都是View，图片-ImageView、文字-TextView、列表-ListView/GridView/RecyclerView、按钮-Button、选择框-CheckBox、输入框-EditText……当然还有更重要的一种是自定义View。除了View，还有ViewGroup，比如LinearLayout、RelativeLayout、FrameLayout，它们是可以包含多个ViewGroup或者View的控件，ViewGroup也继承了View，也就是说View可以是单个控件，也可以是由多个控件组成的一组控件。


## View的位置参数

View的位置由它的四个顶点决定，即View的四个属性：top、left、right、bottom，top表示左上角纵坐标，left表示左上角横坐标，right表示右下角横坐标，bottom表示右下角纵坐标。

> 这些坐标都是相对于View的父容器来说的，所以它是一种相对坐标。


View的这四个参数在源码中对应着mLeft、mRight、mTop和mBottom，获取的方式直接get，比如mLeft=getLeft()。从Android3.0开始View增加了额外几个参数，x、y、translationX、translationY，其中x和y是View左上角的坐标，而translationX和translationY是View左上角相对于父容器的偏移量，这几个参数也是相对于父容器的坐标，且translationX和translationY的初始值是0，它们也有对应的getter/setter方法，这几个参数的换算关系：

```java
x = left + translationX;
y = top + translationY;
```

> View在平移的过程中，left和top表示的是原始左上角的位置信息，值不会发生改变，此时发生改变的是x、y、translationX和translationY。

## 触摸事件MotionEvent

在手指接触屏幕后所产生的一系列事件中，典型的事件类型有以下几种：

- ACTION_DOWN  手指刚接触屏幕时触发该事件
- ACTION_MOVE 手指在屏幕上移动时触发该事件
- ACTION_UP 手指从屏幕抬起时触发该事件

当手指点击屏幕滑动一会然后再抬起这一系列动作，屏幕触发的事件依次是ACTION_DOWN、ACTION_MOVE……、ACTION_UP，只有ACTION_MOVE这个方法会触发多次，其他只会触发一次。通过MotionEvent对象我们可以获取到点击事件的x和y坐标，系统提供了两组方法：getX/getY和getRaw/getRawY，它们的区别是：getX/getY返回的是相对于当前View左上角的x和y，而getRaw/getRawY返回的是相对于手机屏幕左上角的x和y。


## TouchSlop

TouchSlop是系统所能识别出的被认为是滑动的最小距离，默认是8dp，当手指在屏幕上滑动时，如果两次触发ACTION_MOVE事件的距离小于TouchSlop这个常量，那么系统就不会认为是在进行滑动操作。这个常量和设备有关，获取该常量的方式如下：

```java
ViewConfiguration.get(getBaseContext()).getScaledTouchSlop();
```

**该常量的意义**

当我们在处理滑动时，可以利用这个常量来做一些过滤，比如当两次滑动事件的滑动距离小于这个值时我们就可以认为未达到滑动距离的临界值，因此可以不认为是滑动动作，这样会有更好的用户体验。

## VelocityTracker

VelocityTracker表示速度追踪，用于追踪手指在滑动过程中的速度，包括水平和垂直方向的速度，它的使用如下：

在View的onTouchEvent方法中追踪当前单击事件的速度：

```java
VelocityTracker velocityTracker = VelocityTracker.obtain();
velocityTracker.addMovement(event);
velocityTracker.computeCurrentVelocity(1000);
int xVelocity = (int) velocityTracker.getXVelocity();
int yVelocity = (int) velocityTracker.getYVelocity();
```

获取水平方向和竖直方向的速度时，首先要计算速度，即computeCurrentVelocity这个方法必须先调用，才能接着调getXVelocity/getYVelocity。computeCurrentVelocity方法的参数表示的是一个时间单元，单位是ms。另外这里所说的速度指的是一段时间内手指滑过的像素数，比如讲时间设为1000ms时，表示在1s内，手指在水平方向滑动100像素，那么水平速度就是100。

> 速度可以是负数，当手指从下往上滑动或从右往左滑动时速度就为负值。
速度 = （终点位置 - 起点位置）/  时间段


当我们最后不需要再使用VelocityTracker时，要记得调用clear方法来重置并回收内存

```java
velocityTracker.clear();
velocityTracker.recycle();
```

## GestureDetector

GestureDetector表示手势检测，用于辅助检测用户的单击、滑动、长按、双击等动作。使用方法如下：

首先创建一个GestureDetector对象并实现GestureDetector.OnGestureListener接口，根据需要我们还可以实现OnDoubleTapListener接口从而能够监听双击动作。

  ```java
GestureDetector mGestureDetector = new GestureDetector(getBaseContext(), this);
mGestureDetector.setIsLongpressEnabled(false);
```

接着接管目标View的onTouchEvent方法，在待监听View的onTouchEvent方法中添加如下代码：

```java
boolean consume = mGestureDetector.onTouchEvent(event);
return consume;
```

然后我们就可以有选择的实现OnGestureListener和OnDoubleTapListener方法中的方法了。

下面是OnGestureListener接口中的方法：

```java
@Override
public boolean onDown(MotionEvent e) {
    // 手指轻轻触摸屏幕的一瞬间，由一个ACTION_DOWN触发
    return false;
}

@Override
public void onShowPress(MotionEvent e) {
    // 手指轻轻触摸屏幕，尚未松开或拖动，由一个ACTION_DOWN触发，和onDown的区别是手指没有松开或拖动

}

@Override
public boolean onSingleTapUp(MotionEvent e) {
    // 手指轻轻触摸屏幕后松开，伴随着一个ACTION_UP触发，这是单击行为
    return false;
}

@Override
public boolean onScroll(MotionEvent e1, MotionEvent e2, float distanceX, float distanceY) {
    // 手指按下屏幕并拖动，有一个ACTION_DOWN和多个ACTION_MOVE触发，这是拖动行为
    return false;
}

@Override
public void onLongPress(MotionEvent e) {
    // 长按事件

}

@Override
public boolean onFling(MotionEvent e1, MotionEvent e2, float velocityX, float velocityY) {
    // 快速滑动行为，由一个ACTION_DOWN、多个ACTION_MOVE和一个ACTION_UP触发
    return false;
}
```

下面是OnDoubleTapListener接口中的方法：

```
@Override
public boolean onSingleTapConfirmed(MotionEvent e) {
    // 严格的单击行为，如果触发了这个方法，那么不可能再紧跟着另一个单击行为，即这只可能是单击，而不可能是双击中的一次单击
    return false;
}

@Override
public boolean onDoubleTap(MotionEvent e) {
    // 双击，由两次连续的单击组成，该方法不可能和onSingleTapConfirmed同时触发
    return false;
}

@Override
public boolean onDoubleTapEvent(MotionEvent e) {
    // 表示发生了双击行为，在双击的期间，ACTION_DOWN、ACTION_MOVE、ACTION_UP都会触发这个方法
    return false;
}
```

实际开发中，可以不使用GestureDetector，可以自己在View的onTouchEvent方法中实现所需要的监听，如果是监听滑动相关的，还是自己在onTouchEvent方法中实现比较好，如果是单纯的监听某个行为（双击、长按……），就使用GestureDetector。

## Scroller

弹性滑动对象，用于实现View的弹性滑动，当使用View的scrollTo/scrollBy方法来进行滑动时，其过程是瞬间完成的，这个没有过渡效果的滑动用户体验不好。这个时候就可以使用Scroller来实现有过渡效果的滑动，其过程不是在一瞬间完成，而是在一定的时间内完成。Scroller本身无法让View弹性滑动，它需要和View的`computeScroll`方法配合使用才能共同完成这个功能。关键代码如下：

```
mScroller = new Scroller(context);

private void smoothScrollTo(int destX, int destY) {
    int scrollX = getScrollX();
    int delta = destX - scrollX;
    mScroller.startScroll(scrollX, 0, delta, 0, 1000);
    invalidate();
}


@Override
public void computeScroll() {
    if (mScroller.computeScrollOffset()) {
        scrollTo(mScroller.getCurrX(), mScroller.getCurrY());
        postInvalidate();
    }
}
```

以上就是Android系统中关于View的几个重要的基础知识点，不过这也仅仅是View中最基础的东西，接下来View的滑动、事件分发等才是更需要深入理解的知识。

