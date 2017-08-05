---
layout: post
title: View的滑动
category: 技术分享
tags: Android、View
---


目前移动设备流行，我们要在如此小的屏幕上尽可能给用户展现更多的内容，就需要在应用上通过滑动来显示和隐藏部分内容，View作为呈现内容的媒介，具备滑动功能就无可厚非了。


三种方式实现View的滑动：

* 通过View本身提供的scrollTo/scrollBy方法
* 通过动画给View添加平移动画
* 通过改变View的LayoutParams使得View重新布局

#### 通过使用scrollTo/scrollBy方法

先看这两个方法的源码实现

```
/**
 * Set the scrolled position of your view. This will cause a call to
 * {@link #onScrollChanged(int, int, int, int)} and the view will be
 * invalidated.
 * @param x the x position to scroll to
 * @param y the y position to scroll to
 */
public void scrollTo(int x, int y) {
    if (mScrollX != x || mScrollY != y) {
        int oldX = mScrollX;
        int oldY = mScrollY;
        mScrollX = x;
        mScrollY = y;
        invalidateParentCaches();
        onScrollChanged(mScrollX, mScrollY, oldX, oldY);
        if (!awakenScrollBars()) {
            postInvalidateOnAnimation();
        }
    }
}

/**
 * Move the scrolled position of your view. This will cause a call to
 * {@link #onScrollChanged(int, int, int, int)} and the view will be
 * invalidated.
 * @param x the amount of pixels to scroll by horizontally
 * @param y the amount of pixels to scroll by vertically
 */
public void scrollBy(int x, int y) {
    scrollTo(mScrollX + x, mScrollY + y);
}
```

上面的代码看出，scrollBy方法实际上也是调用了scrollTo方法，它实现了基于当前位置的相对滑动，而scrollTo则是实现了基于所传递参数的绝对滑动。在滑动过程中，mScrollX的值总是等于View左边缘和View内容左边缘在水平方向的距离，而mScrollY的值总是等于View上边缘和View内容上边缘在竖直方向的距离，View的边缘指的是View的位置，由四个顶点组成，View内容边缘指的是View中的内容的边缘，scrollTo和scrollBy只能改变View内容的位置而不能改变View在布局中的位置。mScrollX和mScrollY的单位是像素。当View左边缘在View内容左边缘右边时，mScrollX为正值，反之为负值；当View上边缘在View内容上边缘的下边时，mScrollY为正值，反之为负值。

**情景推理**

> 比如我们有一个自定义View，占满整个屏幕，那么这个View的的位置是固定了的，四个顶点分别是屏幕的四个顶点，这个是View怎么滑动都改变不了的，这个场景中View的边缘就是屏幕的四条边，不滑动时，这个View的边缘和View的内容边缘是重合的。当我下拉时，View向下移动，此时View的边缘还是屏幕的四条边，而View内容的上边缘变了。按照上面说的规则，此时View的上边缘在View内容上边缘的上边，即从上往下滑动时，mScrollY是负值，同理推得从左往右滑动时，mScrollX也是负值。


#### 通过使用动画实现

使用平移动画主要是操作View的`translationX`和`translationY`这两个属性，那我们就有两种选择了，传统的补间动画和属性动画，如果使用属性动画，为了兼容Android3.0，需要采用开源动画库nineoldandroids。

> 在3.0以下系统的手机上通过nineoldandroids实现的属性动画本质上还是补间动画。

之前的两篇关于动画的文章中有实例平移动画的，这里就不贴代码了。

记得那时候有个问题不是很理解，就是当使用补间动画来对View做平移或者缩放时，加入View上有点击事件，那么这个有效点击区域还是原来View原始的位置区域，现在更加容易理解了，因为View的尺寸都没改变，如果是使用属性动画，那么这个有效点击区域就变了。

#### 通过改变View的LayoutParams

这种方式其实就是改变View的外边距，也就是margin值：

```
ViewGroup.MarginLayoutParams params = (ViewGroup.MarginLayoutParams) v.getLayoutParams();
params.leftMargin += 10;
params.width += 10;
v.setLayoutParams(params);
// 或者
//v.requestLayout();
```

#### 几种方式的对比

- scrollTo/scrollBy：优点是它是View提供的原生方法，专门用于View的滑动，可以比较方便地实现滑动效果且不影响内部元素的点击事件；缺点是只能滑动View的内容，并不能滑动View本身。

- 动画方式：如果是Android3.0以上且采用属性动画，则没有明显的缺点，但如果采用补间动画或者在Android3.0以下的版本使用属性动画，则不能改变View本身的属性。如果动画元素不需要响应用户的交互，使用动画来滑动比较合适，否则就不太合适。动画有一个很明显的优势就是一些复杂的滑动效果必须使用动画才能实现。

- 改变布局方式：除了使用比较麻烦，没有明显的缺点，主要适用对象是一些具有交互性的View。


Demo：采用动画的方式实现View的全屏滑动

思路就是重写View的onTouchEvent方法，处理其中的ACTION_MOVE事件：

```
private int mLastX, mLastY;

@Override
public boolean onTouchEvent(MotionEvent event) {
    int x = (int) event.getRawX();
    int y = (int) event.getRawY();
    switch (event.getAction()) {
        case MotionEvent.ACTION_DOWN:
            break;
        case MotionEvent.ACTION_MOVE:
            int deltaX = x - mLastX;
            int deltaY = y - mLastY;
            int translationX = (int) ViewHelper.getTranslationX(this) + deltaX;
            int translationY = (int) ViewHelper.getTranslationY(this) + deltaY;
            ViewHelper.setTranslationX(this, translationX);
            ViewHelper.setTranslationY(this, translationY);
            break;
        case MotionEvent.ACTION_UP:
            break;
    }

    mLastX = x;
    mLastY = y;

    return true;
}
```

首先我们通过event.getRawX()和event.getRawY()获取手指当前的坐标，不能使用getX/getY方法，因为是要全屏滑动，所以需要获取当前点击事件在屏幕中的坐标而不是相对于View本身的坐标。然后计算出两次滑动之间的距离，通过nineoldandroids库提供的setTranslationX/setTranslationY方法来实现。

nineoldandroids的jar包下载地址：[https://github.com/JakeWharton/NineOldAndroids/downloads](https://github.com/JakeWharton/NineOldAndroids/downloads)


## 弹性滑动

实现弹性滑动的思路就是将一次大的滑动分成若干次小的滑动在一段时间内完成，实现方式有Scroller、Handler#postDelayed、Thread#sleep等。

#### 使用Scroller


其实上面已经贴过使用Scroller实现滑动的核心代码：

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

当我们构建了一个Scroller并调用其startScroll方法时，Scroller内部其实什么都没做，它只是保存了我们传进去的几个参数：

```
/**
 * Start scrolling by providing a starting point, the distance to travel,
 * and the duration of the scroll.
 * 
 * @param startX Starting horizontal scroll offset in pixels. Positive
 *        numbers will scroll the content to the left.
 * @param startY Starting vertical scroll offset in pixels. Positive numbers
 *        will scroll the content up.
 * @param dx Horizontal distance to travel. Positive numbers will scroll the
 *        content to the left.
 * @param dy Vertical distance to travel. Positive numbers will scroll the
 *        content up.
 * @param duration Duration of the scroll in milliseconds.
 */
public void startScroll(int startX, int startY, int dx, int dy, int duration) {
    mMode = SCROLL_MODE;
    mFinished = false;
    mDuration = duration;
    mStartTime = AnimationUtils.currentAnimationTimeMillis();
    mStartX = startX;
    mStartY = startY;
    mFinalX = startX + dx;
    mFinalY = startY + dy;
    mDeltaX = dx;
    mDeltaY = dy;
    mDurationReciprocal = 1.0f / (float) mDuration;
}
```

这里面startX、startY表示滑动的起点，dx、dy表示要滑动的距离，duration表示滑动的时间，需要注意的是这里的滑动是指View内容的滑动而不是View本身位置的改变。所以仅仅调用startScroll时无法让View滑动的，因为它内部没有做滑动相关的事。使View滑动的真正起作用的代码是invalidate方法，调用invalidate表示要重绘View，在View的draw方法中又会去调用computeScroll方法，computeScroll在View中是一个空实现，因此需要我们自己去动手实现。正是因为这个computeScroll方法，View才能实现弹性滑动。过程是这样的：当View重绘后再draw方法中调用computeScroll方法，而computeScroll又会去向Scroller获取当前的scrollX和scrollY，然后通过scrollTo实现滑动，接着又调用postInvalidate方法再进行一次重绘，和上次一样，使View滑动到新的位置，一直重复这个过程，知道滑动结束。


下面是Scroller的computeScrollOffset方法：

```
/**
 * Call this when you want to know the new location.  If it returns true,
 * the animation is not yet finished.
 */ 
public boolean computeScrollOffset() {
    if (mFinished) {
        return false;
    }

    int timePassed = (int)(AnimationUtils.currentAnimationTimeMillis() - mStartTime);

    if (timePassed < mDuration) {
        switch (mMode) {
        case SCROLL_MODE:
            final float x = mInterpolator.getInterpolation(timePassed * mDurationReciprocal);
            mCurrX = mStartX + Math.round(x * mDeltaX);
            mCurrY = mStartY + Math.round(x * mDeltaY);
            break;
            ...
        }
    }
    else {
        mCurrX = mFinalX;
        mCurrY = mFinalY;
        mFinished = true;
    }
    return true;
}
```

这个方法会根据时间的流逝计算出当前的scrollX和scrollY，计算方法是根据时间流逝的百分比计算出scrollX和scrollY改变的百分比并计算出当前的值，是不是感觉和动画中的插值器类似？没错，这里正是使用了叫`ViscousFluidInterpolator`插值器：代码如下：

```
static class ViscousFluidInterpolator implements Interpolator {
    /** Controls the viscous fluid effect (how much of it). */
    private static final float VISCOUS_FLUID_SCALE = 8.0f;

    private static final float VISCOUS_FLUID_NORMALIZE;
    private static final float VISCOUS_FLUID_OFFSET;

    static {

        // must be set to 1.0 (used in viscousFluid())
        VISCOUS_FLUID_NORMALIZE = 1.0f / viscousFluid(1.0f);
        // account for very small floating-point error
        VISCOUS_FLUID_OFFSET = 1.0f - VISCOUS_FLUID_NORMALIZE * viscousFluid(1.0f);
    }

    private static float viscousFluid(float x) {
        x *= VISCOUS_FLUID_SCALE;
        if (x < 1.0f) {
            x -= (1.0f - (float)Math.exp(-x));
        } else {
            float start = 0.36787944117f;   // 1/e == exp(-1)
            x = 1.0f - (float)Math.exp(1.0f - x);
            x = start + x * (1.0f - start);
        }
        return x;
    }

    @Override
    public float getInterpolation(float input) {
        final float interpolated = VISCOUS_FLUID_NORMALIZE * viscousFluid(input);
        if (interpolated > 0) {
            return interpolated + VISCOUS_FLUID_OFFSET;
        }
        return interpolated;
    }
}
```

这个插值器的实现过程我们先不管，继续看computeScrollOffset方法，当它的返回值是true时表示滑动还未结束，此时还要继续滑动，当返回false时表示滑动结束。

总结：Scroller的工作原理

Scroller本身并不能实现View的滑动，而是要结合View的computeScroll方法才能完成弹性滑动的效果，它不断让View去重绘，而每一次重绘距滑动起始时间会有一个时间间隔，通过这个时间间隔就可以计算出View当前的滑动位置，知道了滑动位置就可以通过scrollTo方法来完成View的滑动。


#### 延时策略

延时策略实现弹性滑动的核心思想就是通过发送一系列延时消息来达到弹性的效果，具体可以使用Handler.postDelayed方法或者Thread.sleep方法。对于postDelayed方法，不断的发送延时消息，就可以在消息中进行View的移动，从而形成弹性滑动，而对于sleep方法，使用while循环不断的滑动View然后sleep，同样可以达到弹性的效果。

#### 使用动画

动画本身就有一个duration属性，相当于它滑动时自带了弹性效果，这部分在上面已经说过了，就不再赘述了。
