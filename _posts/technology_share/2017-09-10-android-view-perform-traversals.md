---
layout: post
title: View的工作流程
category: 技术分享
tags: Android、View
---



### Android中View的层级结构及绘制步骤


之前讲View的事件分发机制时，讲到了DecorView，其实我们要查看一个一个页面的DecorView可以通过Android Studio自带的工具Android Device Monitor，Tools->Android->Android Device Monitor打开，连接手机或者模拟器，打开一个app界面，这时在Android Device Monitor的Device栏我们就可以看到当前连接的设备（真机或模拟器）中安装的app的包名了，选中我们刚才打开的那个app的包名，再点上面的按钮“Dump View Hierarchy for UI Automator”，然后会在右边出现设备上当前界面，再右边就是这个界面对应的布局了，我们看到所有的View都是层级显示，可以很快搞清楚这个界面的结构，而且选中每个节点，可以很清楚的看到这个节点View的信息，所以知道DecorView和android.R.id.content所对应的View就很方便了。

![DecorView层级结构](http://offfjcibp.bkt.clouddn.com/ViewRoot.png)

如图所示，DecorView就是最顶层的那个FrameLayout，它里面包括一个LinearLayout和一个id为statusBarBackground的View，LinearLayout中又包括了一个FrameLayout，注意这里的FrameLayout的y值66，也就是从statusBarBackground的下方开始的，而不是从（0，0）的位置。最后要注意的就是这个content了，当我们在一个Activity中调用setContentView方法时就是将我们的内容View添加到这个id为android.R.id.content的FrameLayout中。

View的绘制是从ViewRootImpl的performTraversals方法开始，方法内部经过performMeasure、performLayout、performDraw三个步骤才能最终将一个View绘制出来，而在这三个方法中又会分别调用顶层View的measure、layout和draw方法，measure负责测量View的宽高，layout负责确定View在父容器中的位置，draw负责绘制内容。

当完成Measure过程后，可以通过getMeasuredWidth和getMeasuredHeight方法来获取到View测量后的宽高。Layout过程完成后，可以得到View的四个顶点位置，通过getLeft、getTop、getRight、getBottom这四个方法。只有当View的draw完成后Veiw的内容才能显示到屏幕上。


### 关于MeasureSpec

MeasureSpec作用与View的测量过程，它可以决定View的测量尺寸，但这并是绝对的，因为可能收到父容器的印象，在测量过程中，系统会将View的LayoutParams根据父容器所施加的规则转换成对应的MeasureSpec，然后再根据这个measureSpec测量出View的宽高。

MeasureSpec实质上是一个32位的int值，高2位表示SpecMode，也就是测量模式，低30位表示SpecSize，SpecSize是在某种测量模式下的规格大小。SpecMode有三种：`UNSPECIFIED`、`EXACTLY`、`AT_MOST`，它们分别表示的意思是：

- UNSPECIFIED：父容器不对View有任何限制。
- EXACTLY：父容器已经测出View所需要的精确大小，这时View的最终大小就是SpecSize所指定的值，当xml中的layout_width和layout_height是match_parent和具体的数值时，测量该View时会使用这中模式。
- AT_MOST：父容器指定了一个可用大小即SpecSize，View的大小不能大于这个值，当xml中的layout_width和layout_height是wrap_content时，测量该View就使用这种模式。

说到这三种模式时，我们不得不说下，当我们要可滑动控件中的内容全部显示时，网上一搜，解决办法基本就是先自定义一个滑动控件，然后重写其onMeasure方法，代码如下：

```
public class ApplyListView extends ListView {

    public ApplyListView(Context context) {
        super(context);
    }


    public ApplyListView(Context context, AttributeSet attrs, int defStyle) {
        super(context, attrs, defStyle);
    }


    public ApplyListView(Context context, AttributeSet attrs) {
        super(context, attrs);
    }


    @Override protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        int expandSpec = MeasureSpec.makeMeasureSpec(Integer.MAX_VALUE >> 2, MeasureSpec.AT_MOST);
        super.onMeasure(widthMeasureSpec, expandSpec);
    }

}
```

估计很多人对onMeasure方法中的代码的意思不理解，下面我来说下，首先看MeasureSpec中的三个方法：

```
// 这个方法的作用是根据大小和模式来生成一个int值，这个int值封装了模式和大小信息
public static int makeMeasureSpec(int size, int mode) 

// 这个方法的作用是通过一个int值来获取里面的模式信息
public static int getMode(int measureSpec)

// 这个方法的作用是通过一个int值来获取里面的大小信息
public static int getSize(int measureSpec)
```

上面我们说过，MeasureSpec是一个32位的int值，高2位表示mode，低30位表示size，而MeasureSpec类中也定义了如下常量：

```
private static final int MODE_SHIFT = 30;
private static final int MODE_MASK  = 0x3 << MODE_SHIFT;
// 0左移30位，也就是int类型的最高两位是00
public static final int UNSPECIFIED = 0 << MODE_SHIFT;
// 1左移30位，也就是int类型的最高两位是01
public static final int EXACTLY     = 1 << MODE_SHIFT;
// 2左移30位，也就是int类型的最高两位是10
public static final int AT_MOST     = 2 << MODE_SHIFT;
```

我们再看看自定义控件中的onMeasure方法，第一句是调用MeasureSpec的makeMeasureSpec方法，这个方法是用来生成一个带有模式和大小信息的int值的，它的两个参数一个是size，一个是mode，size我们传的是Integer.MAX_VALUE >> 2，因为32位int型最大值就是Integer.MAX_VALUE，右移两位，表示后面30位的最大值，而我们在手机上的控件不可能那么大，所以这个模式就得选择MeasureSpec.AT_MOST了。

上面也提到了在测量过程中，系统会将View的LayoutParams根据父容器所施加的规则转换成对应的MeasureSpec，然后再根据这个measureSpec测量出View的宽高，普通View的MeasureSpec是由LayoutParams和父容器共同决定的，对于顶层View也就是DecorView来说，它的MeasureSpec是有窗口的尺寸和它自身的LayoutParams共同决定的。当MeasureSpec确定后就可以在onMeasure中确定View的测量宽高了。


在ViewRootImpl中的measureHierarchy方法中有一段代码：

```
childWidthMeasureSpec = getRootMeasureSpec(baseSize, lp.width);
childHeightMeasureSpec = getRootMeasureSpec(desiredWindowHeight, lp.height);
performMeasure(childWidthMeasureSpec, childHeightMeasureSpec);
```

其作用就是确定DecorView的MeasureSpec的创建过程，其中desiredWindowHeight就是屏幕的高，接着看getRootMeasureSpec方法的实现：

```
private static int getRootMeasureSpec(int windowSize, int rootDimension) {
    int measureSpec;
    switch (rootDimension) {

    case ViewGroup.LayoutParams.MATCH_PARENT:
        // Window can't resize. Force root view to be windowSize.
        measureSpec = MeasureSpec.makeMeasureSpec(windowSize, MeasureSpec.EXACTLY);
        break;
    case ViewGroup.LayoutParams.WRAP_CONTENT:
        // Window can resize. Set max size for root view.
        measureSpec = MeasureSpec.makeMeasureSpec(windowSize, MeasureSpec.AT_MOST);
        break;
    default:
        // Window wants to be an exact size. Force root view to be that size.
        measureSpec = MeasureSpec.makeMeasureSpec(rootDimension, MeasureSpec.EXACTLY);
        break;
    }
    return measureSpec;
}
```

这段代码基本就明确了DecorView的MeasureSpec的创建过程了。当其LayoutParams为MATCH_PARENT时，为精确模式，大小就是屏幕大小；当其LayoutParams为WRAP_CONTENT时，为最大模式，大小不定，但最大不能超过屏幕大小；当其LayoutParams为固定尺寸时，为精确模式，大小就是其指定的大小。

普通View的measure过程是由ViewGroup传递过来的，我们先看看ViewGroup的measureChildWithMargins方法：

```
protected void measureChildWithMargins(View child,
        int parentWidthMeasureSpec, int widthUsed,
        int parentHeightMeasureSpec, int heightUsed) {
    final MarginLayoutParams lp = (MarginLayoutParams) child.getLayoutParams();

    final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
            mPaddingLeft + mPaddingRight + lp.leftMargin + lp.rightMargin
                    + widthUsed, lp.width);
    final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,
            mPaddingTop + mPaddingBottom + lp.topMargin + lp.bottomMargin
                    + heightUsed, lp.height);

    child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
}
```

这个方法中我们可以看到对子View进行measure之前，会通过getChildMeasureSpec方法来计算出子View的MeasureSpec，从几个参数可以看出，子View的MeasureSpec创建与父容器的MeasureSpec和子View本身的LayoutParams有关，还和View的margin、padding有关。

```
public static int getChildMeasureSpec(int spec, int padding, int childDimension) {
    int specMode = MeasureSpec.getMode(spec);
    int specSize = MeasureSpec.getSize(spec);

    int size = Math.max(0, specSize - padding);

    int resultSize = 0;
    int resultMode = 0;

    switch (specMode) {
    // Parent has imposed an exact size on us
    case MeasureSpec.EXACTLY:
        if (childDimension >= 0) {
            resultSize = childDimension;
            resultMode = MeasureSpec.EXACTLY;
        } else if (childDimension == LayoutParams.MATCH_PARENT) {
            // Child wants to be our size. So be it.
            resultSize = size;
            resultMode = MeasureSpec.EXACTLY;
        } else if (childDimension == LayoutParams.WRAP_CONTENT) {
            // Child wants to determine its own size. It can't be
            // bigger than us.
            resultSize = size;
            resultMode = MeasureSpec.AT_MOST;
        }
        break;

    // Parent has imposed a maximum size on us
    case MeasureSpec.AT_MOST:
        if (childDimension >= 0) {
            // Child wants a specific size... so be it
            resultSize = childDimension;
            resultMode = MeasureSpec.EXACTLY;
        } else if (childDimension == LayoutParams.MATCH_PARENT) {
            // Child wants to be our size, but our size is not fixed.
            // Constrain child to not be bigger than us.
            resultSize = size;
            resultMode = MeasureSpec.AT_MOST;
        } else if (childDimension == LayoutParams.WRAP_CONTENT) {
            // Child wants to determine its own size. It can't be
            // bigger than us.
            resultSize = size;
            resultMode = MeasureSpec.AT_MOST;
        }
        break;

    // Parent asked to see how big we want to be
    case MeasureSpec.UNSPECIFIED:
        if (childDimension >= 0) {
            // Child wants a specific size... let him have it
            resultSize = childDimension;
            resultMode = MeasureSpec.EXACTLY;
        } else if (childDimension == LayoutParams.MATCH_PARENT) {
            // Child wants to be our size... find out how big it should
            // be
            resultSize = View.sUseZeroUnspecifiedMeasureSpec ? 0 : size;
            resultMode = MeasureSpec.UNSPECIFIED;
        } else if (childDimension == LayoutParams.WRAP_CONTENT) {
            // Child wants to determine its own size.... find out how
            // big it should be
            resultSize = View.sUseZeroUnspecifiedMeasureSpec ? 0 : size;
            resultMode = MeasureSpec.UNSPECIFIED;
        }
        break;
    }
    //noinspection ResourceType
    return MeasureSpec.makeMeasureSpec(resultSize, resultMode);
}
```

我们看到，确定子View的MeasureSpec，首先是算出父容器除去padding后还剩下的尺寸，然后根据父容器的MeasureSpec和子View的LayoutParams来确定。

- 当父容器的MeasureSpec为EXACTLY时：
    - 如果子View的LayoutParams指定了具体值，那么子View的MeasureSpec为EXACTLY；
    - 如果子View的LayoutParams为MATCH_PARENT，那么子View的MeasureSpec为EXACTLY；
    - 如果子View的LayoutParams为WRAP_CONTENT，那么子View的MeasureSpec为AT_MOST；

- 当父容器的MeasureSpec为AT_MOST时：
    - 如果子View的LayoutParams指定了具体值，那么子View的MeasureSpec为EXACTLY；
    - 如果子View的LayoutParams为MATCH_PARENT，那么子View的MeasureSpec为AT_MOST；
    - 如果子View的LayoutParams为WRAP_CONTENT，那么子View的MeasureSpec为AT_MOST；

- 当父容器的MeasureSpec为UNSPECIFIED时：
    - 如果子View的LayoutParams指定了具体值，那么子View的MeasureSpec为EXACTLY；
    - 如果子View的LayoutParams为MATCH_PARENT，那么子View的MeasureSpec为UNSPECIFIED；
    - 如果子View的LayoutParams为WRAP_CONTENT，那么子View的MeasureSpec为UNSPECIFIED；

> 一般我们开发过程中不会使用到UNSPECIFIED这种模式，它主要用于系统内部多次measure的情形。


### View和ViewGroup的measure过程

如果是原始的View，那么通过measure方法就完成了测量过程，而如果是ViewGroup，除了完成自己的测量外，还需要遍历子元素调用其measure方法，各个子元素再去递归执行这个过程。

先看View的measure方法，这个方法是final类型的，也就是说子类不能重写该方法，而在View的measure方法中会调用onMeasure方法，所以我们只需要看onMeasure的实现即可，在自定义View时，也是重写onMeasure方法。

```
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    setMeasuredDimension(
        getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec), 
        getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
}
```

setMeasuredDimension方法的作用就是设置View的宽高测量值，而getDefaultSize方法就是根据MeasureSpec获取到测量值：

```
public static int getDefaultSize(int size, int measureSpec) {
    int result = size;
    int specMode = MeasureSpec.getMode(measureSpec);
    int specSize = MeasureSpec.getSize(measureSpec);

    switch (specMode) {
    case MeasureSpec.UNSPECIFIED:
        result = size;
        break;
    case MeasureSpec.AT_MOST:
    case MeasureSpec.EXACTLY:
        result = specSize;
        break;
    }
    return result;
}
```

我们主要看最大模式和精确模式这两种情况，最后返回的结果就是specSize这个View的测量值，而View的大小是在layout阶段才最终确定的。无限制模式一般用于系统内容测量，这种情况下，View的大小为getDefaultSize方法的第一个参数size，即宽高分别为getSuggestedMinimumWidth()和getSuggestedMinimumHeight()这两个方法的返回值。这两个方法如下：

```
protected int getSuggestedMinimumWidth() {
    return (mBackground == null) ? mMinWidth : max(mMinWidth, mBackground.getMinimumWidth());
}

protected int getSuggestedMinimumHeight() {
    return (mBackground == null) ? mMinHeight : max(mMinHeight, mBackground.getMinimumHeight());
}
```

这段代码看出，这种模式下View是根据background和minWidth/minHeight这两个属性来决定的，如果没有设置背景，那么返回值就是最小值属性指定的数值，这个值默认为0，也就是当不指定这个属性时，最小值就为0了，如果有背景，返回的是mBackground.getMinimumHeight()，这个方法在Drawable类中，源码如下：

```
public int getMinimumHeight() {
    final int intrinsicHeight = getIntrinsicHeight();
    return intrinsicHeight > 0 ? intrinsicHeight : 0;
}
```

所以这个方法返回的就是背景Drawable的原始宽/高，前提是这个Drawbale有原始宽/高，否则就返回0，比如当背景设置成ShapeDrawable时就没有原始宽高，而BitmapDrawable就有。

从getDefaultSize方法的实现来看，View的宽高由specSize决定，所以，我们自定义View时，需要重写onMeasure方法并设置wrap_content时的自身大小，否则在布局中wrap_content就相当于match_parent，因为当View的layout_width/height为wrap_content时，它的specMode是AT_MOST模式，这种情况下，它的宽高都等于specSzie-padding，也就是父容器当前剩余的空间大小，效果等同于match_parent。那么如何重写onMeasure方法呢？

```
@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {

    super.onMeasure(widthMeasureSpec, heightMeasureSpec);
    int widthMode = MeasureSpec.getMode(widthMeasureSpec);
    int heightMode = MeasureSpec.getMode(heightMeasureSpec);
    int widthSize = MeasureSpec.getSize(widthMeasureSpec);
    int heightSize = MeasureSpec.getSize(heightMeasureSpec);

    if (widthMode == MeasureSpec.AT_MOST && heightMode == MeasureSpec.AT_MOST) {
        setMeasuredDimension(mWidth, mHeight);
    } else if (widthMode == MeasureSpec.AT_MOST) {
        setMeasuredDimension(mWidth, heightMeasureSpec);
    } else {
        setMeasuredDimension(widthMeasureSpec, mHeight);
    }

}
```

我们只需要给View指定一个默认的宽高mWidth和mHeight设置即可。

ViewGroup的measure过程，除了测量自身的大小，还要遍历测量所有子元素的大小，各个子元素也要递归操作这个过程，与View不同的是，ViewGroup是一个抽象类，因此它没有重写View的onMeasure方法，但它提供了一个叫measureChildren的方法：

```
protected void measureChildren(int widthMeasureSpec, int heightMeasureSpec) {
    final int size = mChildrenCount;
    final View[] children = mChildren;
    for (int i = 0; i < size; ++i) {
        final View child = children[i];
        if ((child.mViewFlags & VISIBILITY_MASK) != GONE) {
            measureChild(child, widthMeasureSpec, heightMeasureSpec);
        }
    }
}
```

这个方法就是循环遍历子元素并测量他们的大小，measureChild方法源码如下：

```
protected void measureChild(View child, int parentWidthMeasureSpec,
        int parentHeightMeasureSpec) {
    final LayoutParams lp = child.getLayoutParams();

    final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
            mPaddingLeft + mPaddingRight, lp.width);
    final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,
            mPaddingTop + mPaddingBottom, lp.height);

    child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
}
```

measureChild就是根据子元素的MeasureSpec进行测量，子元素的MeasureSpec通过getChildMeasureSpec方法获得。

之所以ViewGroup没有像View一样统一使用onMeasure方法去测量大小，是因为ViewGroup的子类有不同的布局特性，测量细节也各不相同，ViewGroup也无法做统一实现。

View的measure过程是三大过程中最复杂的一个，measure完成以后，通过getMeasuredWidth/Height方法就可以正确的获取到View测量后的宽高。在某些极端情况下，onMeasure方法中拿到的测量值并不准确，所以最好是在onLayout方法中View的测量宽高的最终值。



现在我们要考虑一个我们开发过程中经常遇到的问题：当Activity启动时，我们想获取某个View的尺寸，以便在达到某个条件时做一些事情，但是打印Log显示这个View的尺寸是0。这是因为View的measure过程和Activity的生命周期应不是同步执行的，要解决这种情况也有好几种办法，下面详细来说一说。

- onWindowFoucsChanged，这个方法的含义是View已经初始化完毕了，宽高已经准备好了。不过这个方法在Activity获得焦点和失去焦点时都会被调用，也就是说如果频繁调用onResume方法和onPause方法，那onWindowFoucsChanged方法也会被频繁调用。

- view.post(runnable)，该方法通过post一个runnable到消息队列尾部，然后等Looper调用此runnable，runnable中即是获取View尺寸的操作。

- ViewTreeObserver，这个类其实是一个对View监听的类，它有很多回调方法，我们使用onGlobalLayoutListener这个接口就可以完成这个功能，当View树的状态发生改变或者View树下面的子View发生变化时，onGlobalLayut方法将会被回调。所以这是一个获取View宽高的时机。


### View的layout过程

layout过程主要是确定自身的位置和确定子元素的位置，当ViewGroup的位置被确定后，它会在onLayout方法中遍历所有子元素并调用其layout方法，在layout方法中onLayout方法又会被调用。

```
public void layout(int l, int t, int r, int b) {
    if ((mPrivateFlags3 & PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT) != 0) {
        onMeasure(mOldWidthMeasureSpec, mOldHeightMeasureSpec);
        mPrivateFlags3 &= ~PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT;
    }

    int oldL = mLeft;
    int oldT = mTop;
    int oldB = mBottom;
    int oldR = mRight;

    boolean changed = isLayoutModeOptical(mParent) ?
            setOpticalFrame(l, t, r, b) : setFrame(l, t, r, b);

    if (changed || (mPrivateFlags & PFLAG_LAYOUT_REQUIRED) == PFLAG_LAYOUT_REQUIRED) {
        onLayout(changed, l, t, r, b);

        if (shouldDrawRoundScrollbar()) {
            if(mRoundScrollbarRenderer == null) {
                mRoundScrollbarRenderer = new RoundScrollbarRenderer(this);
            }
        } else {
            mRoundScrollbarRenderer = null;
        }

        mPrivateFlags &= ~PFLAG_LAYOUT_REQUIRED;

        ListenerInfo li = mListenerInfo;
        if (li != null && li.mOnLayoutChangeListeners != null) {
            ArrayList<OnLayoutChangeListener> listenersCopy =
                    (ArrayList<OnLayoutChangeListener>)li.mOnLayoutChangeListeners.clone();
            int numListeners = listenersCopy.size();
            for (int i = 0; i < numListeners; ++i) {
                listenersCopy.get(i).onLayoutChange(this, l, t, r, b, oldL, oldT, oldR, oldB);
            }
        }
    }

    mPrivateFlags &= ~PFLAG_FORCE_LAYOUT;
    mPrivateFlags3 |= PFLAG3_IS_LAID_OUT;
}
```

layout方法首先会通过setOpticalFrame方法或者setFrame方法来给View设定四个顶点的位置，也就是初始化mLeft、mRight、mTop、mBottom这几个值，一旦四个顶点位置确定，那么View在父容器总的位置也就确定了，接着调用onLayout方法，这个方法的作用是父容器确定子元素的位置，但是View和ViewGroup都没有具体实现这个方法，和onMeasure类似。

之前我们提到过一个问题，View的测量宽高和最终宽高是不是一样的，这个问题其实就是比较View的getMeasuredWidth方法和getWidth方法的返回值是否一样。其实在View的默认实现中，测量宽高和最终宽高是相等的，只不过测量宽高是在measure过程赋值的，而最终宽高则是在layout过程赋值，日常开发中我们可以认为他们相等，但某些特定的情况会导致不一样，比如当我们重写了View的layout方法，改变了其宽高，这时候就不相等了。



### View的draw过程


draw过程作用就是讲View绘制到屏幕上了，draw的步骤有以下几步：

- 绘制背景：background.draw(canvas)
- 绘制自己：onDraw
- 绘制子元素：dispatchDraw
- 绘制装饰：onDrawScrollBars

draw方法源码如下：

```
public void draw(Canvas canvas) {
    final int privateFlags = mPrivateFlags;
    final boolean dirtyOpaque = (privateFlags & PFLAG_DIRTY_MASK) == PFLAG_DIRTY_OPAQUE &&
            (mAttachInfo == null || !mAttachInfo.mIgnoreDirtyState);
    mPrivateFlags = (privateFlags & ~PFLAG_DIRTY_MASK) | PFLAG_DRAWN;

    /*
     * Draw traversal performs several drawing steps which must be executed
     * in the appropriate order:
     *
     *      1. Draw the background
     *      2. If necessary, save the canvas' layers to prepare for fading
     *      3. Draw view's content
     *      4. Draw children
     *      5. If necessary, draw the fading edges and restore layers
     *      6. Draw decorations (scrollbars for instance)
     */

    // Step 1, draw the background, if needed
    int saveCount;

    if (!dirtyOpaque) {
        drawBackground(canvas);
    }

    // skip step 2 & 5 if possible (common case)
    final int viewFlags = mViewFlags;
    boolean horizontalEdges = (viewFlags & FADING_EDGE_HORIZONTAL) != 0;
    boolean verticalEdges = (viewFlags & FADING_EDGE_VERTICAL) != 0;
    if (!verticalEdges && !horizontalEdges) {
        // Step 3, draw the content
        if (!dirtyOpaque) onDraw(canvas);

        // Step 4, draw the children
        dispatchDraw(canvas);

        // Overlay is part of the content and draws beneath Foreground
        if (mOverlay != null && !mOverlay.isEmpty()) {
            mOverlay.getOverlayView().dispatchDraw(canvas);
        }

        // Step 6, draw decorations (foreground, scrollbars)
        onDrawForeground(canvas);

        // we're done...
        return;
    }

    /*
     * Here we do the full fledged routine...
     * (this is an uncommon case where speed matters less,
     * this is why we repeat some of the tests that have been
     * done above)
     */

    boolean drawTop = false;
    boolean drawBottom = false;
    boolean drawLeft = false;
    boolean drawRight = false;

    float topFadeStrength = 0.0f;
    float bottomFadeStrength = 0.0f;
    float leftFadeStrength = 0.0f;
    float rightFadeStrength = 0.0f;

    // Step 2, save the canvas' layers
    int paddingLeft = mPaddingLeft;

    final boolean offsetRequired = isPaddingOffsetRequired();
    if (offsetRequired) {
        paddingLeft += getLeftPaddingOffset();
    }

    int left = mScrollX + paddingLeft;
    int right = left + mRight - mLeft - mPaddingRight - paddingLeft;
    int top = mScrollY + getFadeTop(offsetRequired);
    int bottom = top + getFadeHeight(offsetRequired);

    if (offsetRequired) {
        right += getRightPaddingOffset();
        bottom += getBottomPaddingOffset();
    }

    final ScrollabilityCache scrollabilityCache = mScrollCache;
    final float fadeHeight = scrollabilityCache.fadingEdgeLength;
    int length = (int) fadeHeight;

    // clip the fade length if top and bottom fades overlap
    // overlapping fades produce odd-looking artifacts
    if (verticalEdges && (top + length > bottom - length)) {
        length = (bottom - top) / 2;
    }

    // also clip horizontal fades if necessary
    if (horizontalEdges && (left + length > right - length)) {
        length = (right - left) / 2;
    }

    if (verticalEdges) {
        topFadeStrength = Math.max(0.0f, Math.min(1.0f, getTopFadingEdgeStrength()));
        drawTop = topFadeStrength * fadeHeight > 1.0f;
        bottomFadeStrength = Math.max(0.0f, Math.min(1.0f, getBottomFadingEdgeStrength()));
        drawBottom = bottomFadeStrength * fadeHeight > 1.0f;
    }

    if (horizontalEdges) {
        leftFadeStrength = Math.max(0.0f, Math.min(1.0f, getLeftFadingEdgeStrength()));
        drawLeft = leftFadeStrength * fadeHeight > 1.0f;
        rightFadeStrength = Math.max(0.0f, Math.min(1.0f, getRightFadingEdgeStrength()));
        drawRight = rightFadeStrength * fadeHeight > 1.0f;
    }

    saveCount = canvas.getSaveCount();

    int solidColor = getSolidColor();
    if (solidColor == 0) {
        final int flags = Canvas.HAS_ALPHA_LAYER_SAVE_FLAG;

        if (drawTop) {
            canvas.saveLayer(left, top, right, top + length, null, flags);
        }

        if (drawBottom) {
            canvas.saveLayer(left, bottom - length, right, bottom, null, flags);
        }

        if (drawLeft) {
            canvas.saveLayer(left, top, left + length, bottom, null, flags);
        }

        if (drawRight) {
            canvas.saveLayer(right - length, top, right, bottom, null, flags);
        }
    } else {
        scrollabilityCache.setFadeColor(solidColor);
    }

    // Step 3, draw the content
    if (!dirtyOpaque) onDraw(canvas);

    // Step 4, draw the children
    dispatchDraw(canvas);

    // Step 5, draw the fade effect and restore layers
    final Paint p = scrollabilityCache.paint;
    final Matrix matrix = scrollabilityCache.matrix;
    final Shader fade = scrollabilityCache.shader;

    if (drawTop) {
        matrix.setScale(1, fadeHeight * topFadeStrength);
        matrix.postTranslate(left, top);
        fade.setLocalMatrix(matrix);
        p.setShader(fade);
        canvas.drawRect(left, top, right, top + length, p);
    }

    if (drawBottom) {
        matrix.setScale(1, fadeHeight * bottomFadeStrength);
        matrix.postRotate(180);
        matrix.postTranslate(left, bottom);
        fade.setLocalMatrix(matrix);
        p.setShader(fade);
        canvas.drawRect(left, bottom - length, right, bottom, p);
    }

    if (drawLeft) {
        matrix.setScale(1, fadeHeight * leftFadeStrength);
        matrix.postRotate(-90);
        matrix.postTranslate(left, top);
        fade.setLocalMatrix(matrix);
        p.setShader(fade);
        canvas.drawRect(left, top, left + length, bottom, p);
    }

    if (drawRight) {
        matrix.setScale(1, fadeHeight * rightFadeStrength);
        matrix.postRotate(90);
        matrix.postTranslate(right, top);
        fade.setLocalMatrix(matrix);
        p.setShader(fade);
        canvas.drawRect(right - length, top, right, bottom, p);
    }

    canvas.restoreToCount(saveCount);

    // Overlay is part of the content and draws beneath Foreground
    if (mOverlay != null && !mOverlay.isEmpty()) {
        mOverlay.getOverlayView().dispatchDraw(canvas);
    }

    // Step 6, draw decorations (foreground, scrollbars)
    onDrawForeground(canvas);
}   
```

源码中注释已经很清楚了，通过onDraw方法来draw the content，也就是绘制自身的内容，通过dispatchDraw方法来绘制子元素，这个方法会遍历调用所有子元素的draw方法，draw事件一直传递下去，完成了整个ViewGroup的绘制。

