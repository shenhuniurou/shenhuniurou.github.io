---
layout: post
title: 自定义View-仿虎扑直播比赛界面的打赏按钮
category: 技术分享
tags: 自定义view
---


作为一个资深篮球爱好者，我经常会用虎扑app看比赛直播，后来注意到文字直播界面右下角加了两个按钮，可以在直播过程中送虎扑币，为自己支持的球队加油，具体的效果如下图所示：

![1701061](http://upload-images.jianshu.io/upload_images/1159224-7c7fa3ec3fee2dbc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我个人觉得挺好玩的，所以决定自己实现下这个按钮，废话不多说，先看实现的效果吧：

![hoopview](http://upload-images.jianshu.io/upload_images/1159224-d2787b7f019e3765.gif?imageMogr2/auto-orient/strip)

这个效果看起来和popupwindow差不多，但我是采用自定义view的方式来实现，下面说说过程。

首先从虎扑的效果可以看到，它这两个按钮时浮在整个界面之上的，所以它需要和FrameLayout结合使用，因此我让它的宽度跟随屏幕大小，高度根据dpi固定，它的实际尺寸时这样的：

![1701062](http://upload-images.jianshu.io/upload_images/1159224-e7203f8aab4ada5d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

另外这个view初始化出来我们看到可以分为三块，背景圆、圆内文字、圆上方数字，所以正常状态下，只需要在onDraw方法中画出这三块内容即可。先在初始化方法中将自定义的属性和画笔以及初始化数据准备好：

```java
private void init(Context context, AttributeSet attrs) {
	//获取自定义属性
	TypedArray typedArray = context.obtainStyledAttributes(attrs, R.styleable.HoopView);
	mThemeColor = typedArray.getColor(R.styleable.HoopView_theme_color, Color.YELLOW);
	mText = typedArray.getString(R.styleable.HoopView_text);
	mCount = typedArray.getString(R.styleable.HoopView_count);

	mBgPaint = new Paint();
	mBgPaint.setAntiAlias(true);
	mBgPaint.setColor(mThemeColor);
	mBgPaint.setAlpha(190);
	mBgPaint.setStyle(Paint.Style.FILL);

	mPopPaint = new Paint();
	mPopPaint.setAntiAlias(true);
	mPopPaint.setColor(Color.LTGRAY);
	mPopPaint.setAlpha(190);
	mPopPaint.setStyle(Paint.Style.FILL_AND_STROKE);

	mTextPaint = new TextPaint();
	mTextPaint.setAntiAlias(true);
	mTextPaint.setColor(mTextColor);
	mTextPaint.setTextSize(context.getResources().getDimension(R.dimen.hoop_text_size));

	mCountTextPaint = new TextPaint();
	mCountTextPaint.setAntiAlias(true);
	mCountTextPaint.setColor(mThemeColor);
	mCountTextPaint.setTextSize(context.getResources().getDimension(R.dimen.hoop_count_text_size));

	typedArray.recycle();

	mBigRadius = context.getResources().getDimension(R.dimen.hoop_big_circle_radius);
	mSmallRadius = context.getResources().getDimension(R.dimen.hoop_small_circle_radius);
	margin = (int) context.getResources().getDimension(R.dimen.hoop_margin);
	mHeight = (int) context.getResources().getDimension(R.dimen.hoop_view_height);
	countMargin = (int) context.getResources().getDimension(R.dimen.hoop_count_margin);

	mDatas = new String[] {"1", "10", "100"};
	// 计算背景框改变的长度，默认是三个按钮
	mChangeWidth = (int) (2 * mSmallRadius * 3 + 4 * margin);

}
```

在onMeasure中测出view的宽度后，根据宽度计算出背景圆的圆心坐标和一些相关的数据值。

```java
@Override protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
	int widthSize = MeasureSpec.getSize(widthMeasureSpec);
	mWidth = getDefaultSize(widthSize, widthMeasureSpec);
	setMeasuredDimension(mWidth, mHeight);

	// 此时才测出了mWidth值，再计算圆心坐标及相关值
	cx = mWidth - mBigRadius;
	cy = mHeight - mBigRadius;
	// 大圆圆心
	circle = new PointF(cx, cy);
	// 三个按钮的圆心
	circleOne = new PointF(cx - mBigRadius - mSmallRadius - margin, cy);
	circleTwo = new PointF(cx - mBigRadius - 3 * mSmallRadius - 2 * margin, cy);
	circleThree = new PointF(cx - mBigRadius - 5 * mSmallRadius - 3 * margin, cy);
	// 初始的背景框的边界即为大圆的四个边界点
	top = cy - mBigRadius;
	bottom = cy + mBigRadius;
}
```

因为这里面涉及到点击按钮展开和收缩的过程，所以我定义了如下几种状态，只有在特定的状态下才能进行某些操作。

```java
private int mState = STATE_NORMAL;//当前展开收缩的状态
private boolean mIsRun = false;//是否正在展开或收缩

//正常状态
public static final int STATE_NORMAL = 0;
//按钮展开
public static final int STATE_EXPAND = 1;
//按钮收缩
public static final int STATE_SHRINK = 2;
//正在展开
public static final int STATE_EXPANDING = 3;
//正在收缩
public static final int STATE_SHRINKING = 4;
```

接下来就执行onDraw方法了，先看看代码：

```java
@Override protected void onDraw(Canvas canvas) {
	switch (mState) {
		case STATE_NORMAL:
			drawCircle(canvas);
			break;
		case STATE_SHRINK:
		case STATE_SHRINKING:
			drawBackground(canvas);
			break;
		case STATE_EXPAND:
		case STATE_EXPANDING:
			drawBackground(canvas);
			break;
	}
	drawCircleText(canvas);
	drawCountText(canvas);
}
```

圆上方的数字和圆内的文字是整个过程中一直存在的，所以我将这两个操作放在`switch`之外，正常状态下绘制圆和之前两部分文字，点击展开时绘制背景框展开过程和文字，展开状态下再次点击绘制收缩过程和文字，当然在绘制背景框的方法中也需要不断绘制大圆，大圆也是一直存在的。

上面的绘制方法：

```java
/**
 * 画背景大圆
 * @param canvas
 */
private void drawCircle(Canvas canvas) {
	left = cx - mBigRadius;
	right = cx + mBigRadius;
	canvas.drawCircle(cx, cy, mBigRadius, mBgPaint);
}


/**
 * 画大圆上面表示金币数的文字
 * @param canvas
 */
private void drawCountText(Canvas canvas) {
	canvas.translate(0, -countMargin);
	//计算文字的宽度
	float textWidth = mCountTextPaint.measureText(mCount, 0, mCount.length());
	canvas.drawText(mCount, 0, mCount.length(), (2 * mBigRadius - textWidth - 35) / 2, 0.2f, mCountTextPaint);
}


/**
 * 画大圆内的文字
 * @param canvas
 */
private void drawCircleText(Canvas canvas) {
	StaticLayout layout = new StaticLayout(mText, mTextPaint, (int) (mBigRadius * Math.sqrt(2)), Layout.Alignment.ALIGN_CENTER, 1.0f, 0.0f, true);
	canvas.translate(mWidth - mBigRadius * 1.707f, mHeight - mBigRadius * 1.707f);
	layout.draw(canvas);
	canvas.save();
}


/**
 * 画背景框展开和收缩
 * @param canvas
 */
private void drawBackground(Canvas canvas) {
	left = cx - mBigRadius - mChange;
	right = cx + mBigRadius;
	canvas.drawRoundRect(left, top, right, bottom, mBigRadius, mBigRadius, mPopPaint);
	if ((mChange > 0) && (mChange <= 2 * mSmallRadius + margin)) {
		// 绘制第一个按钮
		canvas.drawCircle(cx - mChange, cy, mSmallRadius, mBgPaint);
		// 绘制第一个按钮内的文字
		canvas.drawText(mDatas[0], cx - (mBigRadius - mSmallRadius) - mChange, cy + 15, mTextPaint);
	} else if ((mChange > 2 * mSmallRadius + margin) && (mChange <= 4 * mSmallRadius + 2 * margin)) {
		// 绘制第一个按钮
		canvas.drawCircle(cx - mBigRadius - mSmallRadius - margin, cy, mSmallRadius, mBgPaint);
		// 绘制第一个按钮内的文字
		canvas.drawText(mDatas[0], cx - mBigRadius - mSmallRadius - margin - 20, cy + 15, mTextPaint);

		// 绘制第二个按钮
		canvas.drawCircle(cx - mChange, cy, mSmallRadius, mBgPaint);
		// 绘制第二个按钮内的文字
		canvas.drawText(mDatas[1], cx - mChange - 20, cy + 15, mTextPaint);
	} else if ((mChange > 4 * mSmallRadius + 2 * margin) && (mChange <= 6 * mSmallRadius + 3 * margin)) {
		// 绘制第一个按钮
		canvas.drawCircle(cx - mBigRadius - mSmallRadius - margin, cy, mSmallRadius, mBgPaint);
		// 绘制第一个按钮内的文字
		canvas.drawText(mDatas[0], cx - mBigRadius - mSmallRadius - margin - 16, cy + 15, mTextPaint);

		// 绘制第二个按钮
		canvas.drawCircle(cx - mBigRadius - 3 * mSmallRadius - 2 * margin, cy, mSmallRadius, mBgPaint);
		// 绘制第二个按钮内的文字
		canvas.drawText(mDatas[1], cx - mBigRadius - 3 * mSmallRadius - 2 * margin - 25, cy + 15, mTextPaint);

		// 绘制第三个按钮
		canvas.drawCircle(cx - mChange, cy, mSmallRadius, mBgPaint);
		// 绘制第三个按钮内的文字
		canvas.drawText(mDatas[2], cx - mChange - 34, cy + 15, mTextPaint);
	} else  if (mChange > 6 * mSmallRadius + 3 * margin) {
		// 绘制第一个按钮
		canvas.drawCircle(cx - mBigRadius - mSmallRadius - margin, cy, mSmallRadius, mBgPaint);
		// 绘制第一个按钮内的文字
		canvas.drawText(mDatas[0], cx - mBigRadius - mSmallRadius - margin - 16, cy + 15, mTextPaint);

		// 绘制第二个按钮
		canvas.drawCircle(cx - mBigRadius - 3 * mSmallRadius - 2 * margin, cy, mSmallRadius, mBgPaint);
		// 绘制第二个按钮内的文字
		canvas.drawText(mDatas[1], cx - mBigRadius - 3 * mSmallRadius - 2 * margin - 25, cy + 15, mTextPaint);

		// 绘制第三个按钮
		canvas.drawCircle(cx - mBigRadius - 5 * mSmallRadius - 3 * margin, cy, mSmallRadius, mBgPaint);
		// 绘制第三个按钮内的文字
		canvas.drawText(mDatas[2], cx - mBigRadius - 5 * mSmallRadius - 3 * margin - 34, cy + 15, mTextPaint);
	}
	drawCircle(canvas);

}
```

然后是点击事件的处理，只有触摸点在大圆内时才会触发展开或收缩的操作，点击小圆时提供了一个接口给外部调用。

```java
@Override public boolean onTouchEvent(MotionEvent event) {
	int action = event.getAction();
	switch (action) {
		case MotionEvent.ACTION_DOWN:
			//如果点击的时候动画在进行，不处理
			if (mIsRun) return true;
			PointF pointF = new PointF(event.getX(), event.getY());
			if (isPointInCircle(pointF, circle, mBigRadius)) { //如果触摸点在大圆内，根据弹出方向弹出或者收缩按钮
				if ((mState == STATE_SHRINK || mState == STATE_NORMAL) && !mIsRun) {
					//展开
					mIsRun = true;//这是必须先设置true，因为onAnimationStart在onAnimationUpdate之后才调用
					showPopMenu();
				} else {
					//收缩
					mIsRun = true;
					hidePopMenu();
				}
			} else { //触摸点不在大圆内
				if (mState == STATE_EXPAND) { //如果是展开状态
					if (isPointInCircle(pointF, circleOne, mSmallRadius)) {
						listener.clickButton(this, Integer.parseInt(mDatas[0]));
					} else if (isPointInCircle(pointF, circleTwo, mSmallRadius)) {
						listener.clickButton(this, Integer.parseInt(mDatas[1]));
					} else if (isPointInCircle(pointF, circleThree, mSmallRadius)) {
						listener.clickButton(this, Integer.parseInt(mDatas[2]));
					}
					mIsRun = true;
					hidePopMenu();
				}
			}
			break;
	}
	return super.onTouchEvent(event);
}
```

展开和收缩的动画是改变背景框的宽度属性的动画，并监听这个属性动画，在宽度值改变的过程中去重新绘制整个view。因为一开始我就确定了大圆小圆的半径和小圆与背景框之间的间距，所以初始化时已经计算好了背景框的宽度：

```java
mChangeWidth = (int) (2 * mSmallRadius * 3 + 4 * margin);
```

```java
/**
 * 弹出背景框
 */
private void showPopMenu() {
	if (mState == STATE_SHRINK || mState == STATE_NORMAL) {
		ValueAnimator animator = ValueAnimator.ofInt(0, mChangeWidth);
		animator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
			@Override public void onAnimationUpdate(ValueAnimator animation) {
				if (mIsRun) {
					mChange = (int) animation.getAnimatedValue();
					invalidate();
				} else {
					animation.cancel();
					mState = STATE_NORMAL;
				}
			}
		});
		animator.addListener(new AnimatorListenerAdapter() {
			@Override public void onAnimationStart(Animator animation) {
				super.onAnimationStart(animation);
				mIsRun = true;
				mState = STATE_EXPANDING;
			}


			@Override public void onAnimationCancel(Animator animation) {
				super.onAnimationCancel(animation);
				mIsRun = false;
				mState = STATE_NORMAL;
			}


			@Override public void onAnimationEnd(Animator animation) {
				super.onAnimationEnd(animation);
				mIsRun = false;
				//动画结束后设置状态为展开
				mState = STATE_EXPAND;
			}
		});
		animator.setDuration(500);
		animator.start();
	}
}
```

```java
/**
 * 隐藏弹出框
 */
private void hidePopMenu() {
	if (mState == STATE_EXPAND) {
		ValueAnimator animator = ValueAnimator.ofInt(mChangeWidth, 0);
		animator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
			@Override public void onAnimationUpdate(ValueAnimator animation) {
				if (mIsRun) {
					mChange = (int) animation.getAnimatedValue();
					invalidate();
				} else {
					animation.cancel();
				}
			}
		});
		animator.addListener(new AnimatorListenerAdapter() {
			@Override public void onAnimationStart(Animator animation) {
				super.onAnimationStart(animation);
				mIsRun = true;
				mState = STATE_SHRINKING;
			}


			@Override public void onAnimationCancel(Animator animation) {
				super.onAnimationCancel(animation);
				mIsRun = false;
				mState = STATE_EXPAND;
			}


			@Override public void onAnimationEnd(Animator animation) {
				super.onAnimationEnd(animation);
				mIsRun = false;
				//动画结束后设置状态为收缩
				mState = STATE_SHRINK;
			}
		});
		animator.setDuration(500);
		animator.start();
	}
}
```

这个过程看起来是弹出或收缩，实际上宽度值每改变一点，就将所有的组件重绘一次，只是文字和大圆等内容的尺寸及位置都没有变化，只有背景框的宽度值在变，所以才有这种效果。

在xml中的使用：

```xml
<LinearLayout
	android:layout_width="match_parent"
	android:layout_height="wrap_content"
	android:layout_alignParentBottom="true"
	android:layout_marginBottom="20dp"
	android:layout_alignParentRight="true"
	android:orientation="vertical">

	<com.xx.hoopcustomview.HoopView
		android:id="@+id/hoopview1"
		android:layout_width="match_parent"
		android:layout_height="wrap_content"
		android:layout_marginRight="10dp"
		app:text="支持火箭"
		app:count="1358"
		app:theme_color="#31A129"/>

	<com.xx.hoopcustomview.HoopView
		android:id="@+id/hoopview2"
		android:layout_width="match_parent"
		android:layout_height="wrap_content"
		android:layout_marginRight="10dp"
		app:text="热火无敌"
		app:count="251"
		app:theme_color="#F49C11"/>
</LinearLayout>
```

activity中使用：

```java
hoopview1 = (HoopView) findViewById(R.id.hoopview1);
hoopview1.setOnClickButtonListener(new HoopView.OnClickButtonListener() {
	@Override public void clickButton(View view, int num) {
		Toast.makeText(MainActivity.this, "hoopview1增加了" + num, Toast.LENGTH_SHORT).show();
	}
});
```


大致实现过程就是这样，与原始效果还是有点区别，我这个还有很多瑕疵，比如文字的位置居中问题，弹出或收缩时，小圆内的文字的旋转动画我没有实现。

代码地址：
[https://github.com/shenhuniurou/HoopCustomView](https://github.com/shenhuniurou/HoopCustomView)