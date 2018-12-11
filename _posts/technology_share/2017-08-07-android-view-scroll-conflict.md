---
layout: post
title: View滑动冲突及解决方案
category: blog
tags: Android、View
---



当父容器和子元素都可以滑动时，就会产生滑动冲突，比如ScrollView里面套一个ListView或者GridView，或者ListView中嵌套了ViewPager等等。

常见的滑动冲突场景有以下几种：

- 外部ViewGroup的滑动方向和内部元素滑动方向不一致
- 外部滑动方向和内部滑动方法一致
- 以上两种情况的嵌套

第一种场景比如讲ViewPager和Fragment配置使用所组成的页面滑动效果。这种效果通过左右滑动来切换页面，而每个页面的内部往往又是一个竖直滑动的ListView，本来这种情况是有滑动冲突的，但是ViewPager内部处理了这种冲突。如果才使用ScrollView而不是ViewPager，那么我们就必须手动来处理滑动冲突了，否则出现的后果就是内外两层只有一层能够滑动，

第二种场景当内外两层在同一方向滑动时，如果手指触摸开始滑动，但是系统无法知道到底要让哪一层滑动，所以会出现这时候会出问题。

#### 滑动冲突的处理规则

对于场景1的处理规则：当手指左右滑动时，需要让外部的元素拦截点击事件，当手指上下滑动时，需要让内部的元素拦截点击事件，这时候就可以根据元素的特性来解决冲突问题。具体一点就是根据滑动方向的不同来判断到底由谁来拦截事件。如果是斜着滑动的，那我们可以计算出两点之间的水平距离和竖直距离，距离较大的那个方向判定为滑动方向，然后决定是拦截对应的元素。

对于场景2和场景3的处理规则：这种情况无法根据滑动的角度距离差等因素来判断。但是我们可以根据不同的状态来决定滑动哪个元素。

解决滑动冲突的方法：

**拦截父容器**

外部拦截是指所有的点击事件都先经过父容器的拦截处理，如果父容器需要这个事件就拦截，不需要就不拦截。外部拦截需要重写父容器的onInterceptTouchEvent方法。

```
public boolean onInterceotTouchEvent(MotionEvent event) {
    
    boolean intercepted = false;
    int x = (int) event.getX();
    int y = (int) evnet.getY();

    switch(event.getAction) {

        case MotionEvent.ACTION_DOWN:
            intercepted = false;
            break;

        case MotionEvent.ACTION_MOVE:

            if (父容器是否需要处理当前点击事件) {

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
    return intercepted;

}
```

以上就是外部拦截的伪代码，在ACTION_DOWN事件中，父容器必须返回false，因为父容器一旦拦截了ACTION_DOWN事件，那么后续一系列的ACTION_MOVE和ACTION_UP事件都会直接交给父容器处理，此时事件就没办法再传递给子元素了，ACTION_UP一般也不需要拦截，本身也没多大意义，ACTION_MOVE根据需要来决定是否拦截。

**拦截子控件**

内部拦截是指父容器不拦截任何事件，所有的事件都交给子元素，如果子元素需要此事件就直接消耗，否则就交给父容器来处理，这种方法和Android中事件分发机制不一致，需要配合requestDisallowInterceptTouchEvent方法才能正常工作。内部拦截需要重写子元素的dispatchTouchEvent方法：

```
public boolean dispatchTouchEvent(MotionEvent event) {
    
    int x = (int) event.getX();
    int y = (int) evnet.getY();

    switch(event.getAction) {

        case MotionEvent.ACTION_DOWN:
            parent.requestDisallowInterceptTouchEvent(true);
            break;

        case MotionEvent.ACTION_MOVE:

            int deltaX = x - mLastX;
            int deltaY = y - mLastY;

            if (父容器需要此类点击事件) {
                parent.requestDisallowInterceptTouchEvent(false);
            }

            break;

        case MotionEvent.ACTION_UP:

            break;
    }

    mLastX = x;
    mLastY = y;
    return super.dispatchTouchEvent(evnet);

}
```

除了子元素需要做上述所示的修改外，父元素也要默认拦截除了ACTION_DOWN以外的其它事件，这样当子元素调用parent.requestDisallowInterceptTouchEvent(false);时父容器才能继续拦截所需要的事件。

之所以父容器不能拦截ACTION_DOWN，是因为ACTION_DOWN事件不受FLAG_DISALLOW_INTERCEPT这个标记位的影响，一旦父容器拦截了ACTION_DOWN，那么所有的事件都无法传递到子元素，，内部拦截就无效了。父容器要做的修改如下代码所示：

```
public boolean onInterceptTouchEvent(MotionEvent event) {
    if(evnet.getAction == MotionEvent.ACTION_DOWN) {
        return false;    
    } else {
        return true;
    }
}
```

接下来通过一个demo来分别实现通过外部拦截和内部拦截来解决滑动冲突。

我们先定义一个可以水平滑动的ViewGroup，其实就是仿ViewPager的功能，然后在里面添加若干个竖直滑动的ListView，这就模拟了一个典型的滑动冲突的场景。根据滑动策略，我们可以选择水平和竖直滑动距离差来解决滑动冲突。

```
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
                    // 如果父容器滑动没有结束，那么下一个事件仍然交给父容器来处理，
                    // 避免当父容器水平滑动没有结束时，用户立即竖直滑动，而造成界面停留在中间状态这个不好的体验。
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
        } else if (heightSpecMode == MeasureSpec.AT_MOST) {
            final View childView = getChildAt(0);
            measuredHeight = childView.getMeasuredHeight();
            setMeasuredDimension(widthSpaceSize, childView.getMeasuredHeight());
        } else if (widthSpecMode == MeasureSpec.AT_MOST) {
            final View childView = getChildAt(0);
            measuredWidth = childView.getMeasuredWidth() * childCount;
            setMeasuredDimension(measuredWidth, heightSpaceSize);
        } else {
            final View childView = getChildAt(0);
            measuredWidth = childView.getMeasuredWidth() * childCount;
            measuredHeight = childView.getMeasuredHeight();
            setMeasuredDimension(measuredWidth, measuredHeight);
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

content_layout布局：

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <TextView
        android:id="@+id/title"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginBottom="5dp"
        android:layout_marginTop="5dp"
        android:textColor="@android:color/white" />

    <ListView
        android:id="@+id/list"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="#fff4f7f9"
        android:cacheColorHint="#00000000"
        android:divider="#dddbdb"
        android:dividerHeight="1px"
        android:listSelector="@android:color/transparent" />

</LinearLayout>
```

在Activity的layout中加入我们自定义的ViewGroup：

```
<com.shenhuniurou.viewdemo.CustomViewPager
    android:id="@+id/container"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@android:color/holo_red_light">

</com.shenhuniurou.viewdemo.CustomViewPager>
```

Activity中的主要代码：

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
        textView.setText("这是第 " + (i + 1) + "页");
        layout.setBackgroundColor(Color.rgb(255 / (i + 10), 255 / (i + 10), 10));
        addListView(layout);
        mListContainer.addView(layout);
    }
}

private void addListView(ViewGroup layout) {
    ListView listView = (ListView) layout.findViewById(R.id.list);
    List<String> datas = new ArrayList<>();
    for (int i = 0; i < 50; i++) {
        datas.add("item " + i);
    }

    ArrayAdapter<String> adapter = new ArrayAdapter<>(this, R.layout.content_list_item, R.id.name, datas);
    listView.setAdapter(adapter);
    listView.setOnItemClickListener(new AdapterView.OnItemClickListener() {
        @Override
        public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
            Toast.makeText(MainActivity.this, "click item", Toast.LENGTH_SHORT).show();
        }
    });
}
```

效果如图：

![拦截父容器解决滑动冲突效果示例](http://upload-images.jianshu.io/upload_images/1159224-e014824e0df187f0.gif?imageMogr2/auto-orient/strip)

上面介绍了使用拦截父容器的方法，如果要从拦截子控件入手，又该怎么做呢？其实也很简单，原理就是上面所说的重写ListView的dispatchTouchEvent方法，让父容器拦截掉ACTION_MOVE事件，判断水平方向和竖直方向滑动的距离谁大，如果是水平方向滑动距离大，父容器就拦截MOVE事件。

```
@Override
public boolean dispatchTouchEvent(MotionEvent event) {
    int x = (int) event.getX();
    int y = (int) event.getY();

    switch (event.getAction()) {
        case MotionEvent.ACTION_DOWN: {
            // 让父容器不拦截ACTION_DOWN事件
            customViewPager.requestDisallowInterceptTouchEvent(true);
            break;
        }
        case MotionEvent.ACTION_MOVE: {
            int deltaX = x - mLastX;
            int deltaY = y - mLastY;
            if (Math.abs(deltaX) > Math.abs(deltaY)) {
                // 如果水平方向滑动的距离多一点，那就表示让父容器水平滑动，子控件不滑动，让父容器拦截事件
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
```

另外为了避免父容器水平方向滑动还未停止但用户立即开始竖直滑动时，父容器里面的子控件可能会停留在一个中间状态的情况，当水平方法还在滑动时，让父容器拦截事件去处理，所以这里还是要重写父容器的onInterceptTouchEvent方法：

```
@Override
public boolean onInterceptTouchEvent(MotionEvent ev) {

    if (ev.getAction() == MotionEvent.ACTION_DOWN) {
        if (!mScroller.isFinished()) {
            mScroller.abortAnimation();
            return true;
        }
        return false;
    } else {
        return true;
    }
}
```


以上就是解决外部ViewGroup的滑动方向和内部元素滑动方向不一致这种场景的两种方法了。如果是内外元素滑动方向一致呢，又该怎么解决？

这种情况解决办法和上面的场景一样，只是滑动规则不同而已，比如当使用拦截父容器的方法时，就是在MOTIONEVENT.ACTION_MOVE事件中的判断条件不同，根据我们自己的需求来定制，看当前事件要交给父容器还是子元素来处理。也就是说上面那两段伪代码基本就是解决滑动冲突的通用方案了。

