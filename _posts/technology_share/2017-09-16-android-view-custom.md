---
layout: post
title: 自定义View
category: blog
tags: Android、View
---



自定义View有多种形式，可以继承自View，也可以继承自ViewGroup，还可以直接继承Andorid系统现有的View组件，比如TextView、ImageView、LinearLayout等，且每种方式都有它的使用场景。

- 继承View
这种方式需要重写onDraw方法，主要用于实现一些不规则的布局效果，通过xml布局不容易实现的情况下使用该方式，采用这种方式需要我们自己支持wrap_content并且处理padding等。

- 继承ViewGroup
该方式主要用于实现自定义的布局，把几个View重新组合在一起，形成一个整体的View。这种方式比较复杂，需要合适地处理ViewGroup的测量和布局这两个过程，并同时处理子元素的测量和布局过程。

- 继承系统现有的View
这种方式适用于扩展现有View的功能，比较常见，开发者不需要自己处理wrap_content并且处理padding等。

- 继承系统现有的ViewGroup
和方法2类似，但方法2更接近View的底层实现。

#### 自定义View的注意事项

1、直接继承View或者ViewGroup的控件，如果不在onMeasure方法中对wrao_content做特出处理，那么当外界在布局中使用wrap_content时就无法达到预期的效果，必须要让这种方式自定义的View支持wrap_content。

2、另外，还需要处理padding，如果不处理padding，那么padding属性在这个自定义View中是不起作用的，如果是继承自ViewGroup，还需要在onMeasure和onLayout方法中考虑padding和子元素的margin对布局造成的影响，否则自定义View的padding和子元素的margin会不起作用。

3、当自定义View中需要停止线程或者动画时，可以在onDetachedFromWindow方法中执行停止动画或线程的操作。因为当包含此View的Activity退出或者当前View被remove时，View的onDetachedFromWindow方法会被调用，和此方法对应的是onAttachedToWindow，当包含此View的Activity启动时，View的onAttachedToWindow方法会被调用。且当View变得不可见时我们也需要停止线程和动画，否则可能会造成内存泄漏。


4、当自定义View带有滑动嵌套情形时，要处理好滑动冲突，否则会影响View的效果。

5、在自定义View的内部最好不要使用Handler，因为它内部已经提供了post系统的方法，可完全代替Handler的作用。


针对第一二点，需要处理支持wrap_content的情况时，我们可重写onMeasure方法。

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

当然我们需要先定义mWidth和mHeight这两个宽高默认值，这个就随便定义了。接下来处理padding的情况，也就是计算时我们要考虑四个padding值，比如画圆心时，需要这样写：

```
@Override
protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);

    int paddingLeft = getPaddingLeft();
    int paddingRight = getPaddingRight();
    int paddingTop = getPaddingTop();
    int paddingBottom = getPaddingBottom();

    int width = getWidth() - paddingLeft - paddingRight;
    int height = getHeight() - paddingBottom - paddingTop;

    // 大圆半径
    int radius = Math.min(width, height) / 2;
    canvas.drawCircle(paddingLeft + width / 2, paddingTop + height / 2, radius, borderPaint);

    // 小圆半径
    int smallRadius = radius - (int) borderWidth;
    canvas.drawCircle(paddingLeft + width / 2, paddingTop + height / 2, smallRadius, mPaint);
}
```

开发者还可以根据自己的需求为自定义View添加自定义属性，一般在xml中属性以android开头的都是系统自带的属性，添加自定义属性有如下几个步骤：

首先，在values文件夹下创建自定义属性的xml，一般是attrs.xml，然后在这个xml文件中定义自定义属性集合名，集合中就可以自定义多个属性了。比如：

```
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <declare-styleable name="CircleView">

        <attr name="circle_color" format="color" />
        <attr name="border_color" format="color" />
        <attr name="border_width" format="dimension" />
    </declare-styleable>

</resources>
```

在自定义View的初始化方法中需要获取到这几个自定义的属性:

```
public class CircleView extends View {


    private Paint mPaint, borderPaint;
    private int mWidth, mHeight;
    private int circleColor, borderColor;
    private float borderWidth;

    public CircleView(Context context) {
        this(context, null);
    }

    public CircleView(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public CircleView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init(context, attrs);
    }

}
```

```
private void init(Context context, AttributeSet attrs) {
    TypedArray a = context.obtainStyledAttributes(attrs, R.styleable.CircleView);
    circleColor = a.getColor(R.styleable.CircleView_circle_color, Color.GREEN);
    borderColor = a.getColor(R.styleable.CircleView_border_color, Color.WHITE);
    borderWidth = a.getDimension(R.styleable.CircleView_border_width, 0);
    a.recycle();

    mPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
    mPaint.setColor(circleColor);

    borderPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
    borderPaint.setColor(borderColor);

    mWidth = mHeight = 200;
}
```

最后在xml中使用自定义View并添加自定义属性：

```
<com.shenhuniurou.viewdemo.CircleView
        android:id="@+id/circleView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_margin="20dp"
        android:background="@color/colorAccent"
        android:padding="10dp"
        app:border_color="@android:color/holo_blue_light"
        app:border_width="5dp"
        app:circle_color="@android:color/holo_orange_light" />
```

效果如图所示：

![](http://upload-images.jianshu.io/upload_images/1159224-c5f71bcd22eea889.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


> 我们知道在xml中使用自定义属性时，前缀是app:，而且必须在布局文件开头添加schemas声明：xmlns:app="http://schemas.android.com/apk/res-auto"，当然这个前缀app可以自定义，但必须保证这个前缀和布局中自定义属性的前缀保持一致。比如你的声明是xmlns:shenhuniurou="http://schemas.android.com/apk/res-auto"，那么自定义属性就得这么写：shenhuniurou:border_width="5dp"。


以上是我们基于View来继承的自定义View的实现，下面我们来试试继承ViewGroup的自定义View。

```
package com.shenhuniurou.viewdemo;

import android.content.Context;
import android.util.AttributeSet;
import android.view.MotionEvent;
import android.view.VelocityTracker;
import android.view.View;
import android.view.ViewGroup;
import android.widget.Scroller;

/**
 * Created by Daniel on 2017/8/5.
 */

public class CustomViewPager extends ViewGroup {

    private Scroller mScroller;
    private VelocityTracker mVelocityTracker;

    private int mChildWidth;// 子View的宽度
    private int mChildIndex;// 子View的位置索引
    private int mChildrenSize;// 子View个数

    private int mLastXIntercept = 0;
    private int mLastYIntercept = 0;

    private int mLastX = 0;
    private int mLastY = 0;


    public CustomViewPager(Context context) {
        this(context, null);
    }

    public CustomViewPager(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public CustomViewPager(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init();
    }

    private void init() {
        mScroller = new Scroller(getContext());
        mVelocityTracker = VelocityTracker.obtain();
    }

    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        boolean intercepted = false;
        int x = (int) ev.getX();
        int y = (int) ev.getY();

        switch (ev.getAction()) {
            case MotionEvent.ACTION_DOWN:
                intercepted = false;
                if (!mScroller.isFinished()) {
                    mScroller.abortAnimation();
                    intercepted = true;
                }
                break;
            case MotionEvent.ACTION_MOVE:
                int deltaX = x - mLastXIntercept;
                int deltaY = y - mLastYIntercept;

                if (Math.abs(deltaX) > Math.abs(deltaY)) {
                    // 表示水平滑动，父容器需要拦截
                    intercepted = true;
                } else {
                    intercepted = false;
                }
                break;
            case MotionEvent.ACTION_UP:
                intercepted = false;
                break;
        }
        mLastX = x;
        mLastY = y;
        mLastXIntercept = x;
        mLastYIntercept = y;

        return intercepted;
    }


    @Override
    public boolean onTouchEvent(MotionEvent event) {
        mVelocityTracker.addMovement(event);

        int x = (int) event.getX();
        int y = (int) event.getY();

        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                if (!mScroller.isFinished()) {
                    mScroller.abortAnimation();
                }
                break;
            case MotionEvent.ACTION_MOVE:

                int deltaX = x - mLastX;
                int deltaY = y - mLastY;
                scrollBy(-deltaX, 0);

                break;
            case MotionEvent.ACTION_UP:

                int scrollX = getScrollX();

                mVelocityTracker.computeCurrentVelocity(1000);
                // 计算水平速度
                float xVelocity = mVelocityTracker.getXVelocity();

                // 这里了是模拟ViewPager快速滑动时，即使只滑动了一小段距离，也可以滑到下一页去
                if (Math.abs(xVelocity) >= 50) {
                    mChildIndex = xVelocity > 0 ? mChildIndex - 1 : mChildIndex + 1;
                } else {
                    mChildIndex = (scrollX + mChildWidth / 2) /mChildWidth;
                }
                mChildIndex = Math.max(0, Math.min(mChildIndex, mChildrenSize - 1));

                int dx = mChildIndex * mChildWidth - scrollX;

                mScroller.startScroll(getScrollX(), 0, dx, 0, 500);
                invalidate();

                mVelocityTracker.clear();

                break;
        }

        mLastX = x;
        mLastY = y;

        return true;
    }

    @Override
    public void computeScroll() {
        if (mScroller.computeScrollOffset()) {
            scrollTo(mScroller.getCurrX(), mScroller.getCurrY());
            postInvalidate();
        }
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        int measuredWidth = 0;
        int measuredHeight = 0;
        final int childCount = getChildCount();

        measureChildren(widthMeasureSpec, heightMeasureSpec);

        int widthSpaceSize = MeasureSpec.getSize(widthMeasureSpec);
        int widthSpecMode = MeasureSpec.getMode(widthMeasureSpec);
        int heightSpaceSize = MeasureSpec.getSize(heightMeasureSpec);
        int heightSpecMode = MeasureSpec.getMode(heightMeasureSpec);
        if (childCount == 0) {
            setMeasuredDimension(0, 0);
        } else if (widthSpecMode == MeasureSpec.AT_MOST && heightSpecMode == MeasureSpec.AT_MOST) {
            final View childView = getChildAt(0);
            measuredWidth = childView.getMeasuredWidth() * childCount;
            measuredHeight = childView.getMeasuredHeight();
            setMeasuredDimension(measuredWidth, measuredHeight);
        } else if (heightSpecMode == MeasureSpec.AT_MOST) {
            final View childView = getChildAt(0);
            measuredHeight = childView.getMeasuredHeight();
            setMeasuredDimension(widthSpaceSize, measuredHeight);
        } else if (widthSpecMode == MeasureSpec.AT_MOST) {
            final View childView = getChildAt(0);
            measuredWidth = childView.getMeasuredWidth() * childCount;
            setMeasuredDimension(measuredWidth, heightSpaceSize);
        }
    }

    @Override
    protected void onLayout(boolean changed, int l, int t, int r, int b) {
        int childLeft = 0;
        final int childCount = getChildCount();
        mChildrenSize = childCount;

        for (int i = 0; i < childCount; i++) {
            final View childView = getChildAt(i);
            if (childView.getVisibility() != View.GONE) {
                final int childWidth = childView.getMeasuredWidth();
                mChildWidth = childWidth;
                childView.layout(childLeft, 0, childLeft + childWidth, childView.getMeasuredHeight());
                childLeft += childWidth;
            }
        }
    }


    @Override
    protected void onDetachedFromWindow() {
        mVelocityTracker.recycle();
        super.onDetachedFromWindow();
    }


}

```

```
package com.shenhuniurou.viewdemo;

import android.content.Context;
import android.util.AttributeSet;
import android.view.MotionEvent;
import android.widget.ListView;

/**
 * Created by Daniel on 2017/8/5.
 */

public class CustomListView extends ListView {

    private CustomViewPager customViewPager;

    // 分别记录上次滑动的坐标
    private int mLastX = 0;
    private int mLastY = 0;

    public CustomListView(Context context) {
        this(context, null);
    }

    public CustomListView(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public CustomListView(Context context, AttributeSet attrs, int defStyle) {
        super(context, attrs, defStyle);
    }

    public void setCustomViewPager(CustomViewPager customViewPager) {
        this.customViewPager = customViewPager;
    }

    @Override
    public boolean dispatchTouchEvent(MotionEvent event) {
        int x = (int) event.getX();
        int y = (int) event.getY();

        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN: {
                customViewPager.requestDisallowInterceptTouchEvent(true);
                break;
            }
            case MotionEvent.ACTION_MOVE: {
                int deltaX = x - mLastX;
                int deltaY = y - mLastY;
                if (Math.abs(deltaX) > Math.abs(deltaY)) {
                    customViewPager.requestDisallowInterceptTouchEvent(false);
                }
                break;
            }
            case MotionEvent.ACTION_UP: {
                break;
            }
            default:
                break;
        }

        mLastX = x;
        mLastY = y;
        return super.dispatchTouchEvent(event);
    }

}
```

```
<com.shenhuniurou.viewdemo.CustomViewPager
    android:id="@+id/container"
    android:layout_width="match_parent"
    android:layout_height="match_parent">


</com.shenhuniurou.viewdemo.CustomViewPager>
```

```
private void initView() {
    LayoutInflater inflater = getLayoutInflater();
    mListContainer = (CustomViewPager) findViewById(R.id.container);
    int screenWidth = MyUtils.getScreenMetrics(this).widthPixels;

    // 往ViewGroup中添加ListView，这里是把含有ListView的一整个布局加进去
    for (int i = 0; i < 5; i++) {
        ViewGroup layout = (ViewGroup) inflater.inflate(R.layout.content_layout, mListContainer, false);
        layout.getLayoutParams().width = screenWidth;
        TextView textView = (TextView) layout.findViewById(R.id.title);
        textView.setText("第 " + (i + 1) + "页");
        layout.setBackgroundColor(Color.rgb(255 / (i + 10), 255 / (i + 10), 10));
        addListView(layout);
        mListContainer.addView(layout);
    }
}

private void addListView(ViewGroup layout) {
    CustomListView listView = (CustomListView) layout.findViewById(R.id.list);
    List<String> datas = new ArrayList<>();
    for (int i = 0; i < 50; i++) {
        datas.add("item " + i);
    }

    ArrayAdapter<String> adapter = new ArrayAdapter<>(this, R.layout.content_list_item, R.id.name, datas);
    listView.setCustomViewPager(mListContainer);
    listView.setAdapter(adapter);
    listView.setOnItemClickListener(new AdapterView.OnItemClickListener() {
        @Override
        public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
            Toast.makeText(MainActivity.this, "click item", Toast.LENGTH_SHORT).show();

        }
    });
}
```

主要关注的就是onMeasure方法中对于宽高是wrap_content的处理和onLayout方法中对子元素的布局，CustomListView是为了解决横向滑动冲突而自定义的ListView，最后的效果图：

![](http://upload-images.jianshu.io/upload_images/1159224-bc61c328727c963f.gif?imageMogr2/auto-orient/strip)


当然这都是一些最基本的自定义View，想要达到随心所欲的自定义View的境界，推荐看这两个系列：

- [HenCoder Android 开发进阶: 自定义 View 1-1 绘制基础](http://hencoder.com/ui-1-1/)
- [HenCoder Android 开发进阶: 自定义 View 1-2 Paint 详解](http://hencoder.com/ui-1-2/)
- [HenCoder Android 开发进阶：自定义 View 1-3 drawText() 文字的绘制](http://hencoder.com/ui-1-3/)
- [HenCoder Android 开发进阶：自定义 View 1-4 Canvas 对绘制的辅助 clipXXX() 和 Matrix](http://hencoder.com/ui-1-4/)
- [HenCoder Android 开发进阶：自定义 View 1-5 绘制顺序](http://hencoder.com/ui-1-5/)
- [HenCoder Android 自定义 View 1-6：属性动画 Property Animation（上手篇）](http://hencoder.com/ui-1-6/)
- [HenCoder Android 自定义 View 1-7：属性动画 Property Animation（进阶篇）](http://hencoder.com/ui-1-7/)

- [安卓自定义View教程目录](http://www.gcssloop.com/customview/CustomViewIndex)

