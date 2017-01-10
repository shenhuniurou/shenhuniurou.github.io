---
layout: post
title: �Զ���View-�»���ֱ����������Ĵ��Ͱ�ť
category: ��������
tags: �Զ���view
---


��Ϊһ���������򰮺��ߣ��Ҿ������û���app������ֱ��������ע�⵽����ֱ���������½Ǽ���������ť��������ֱ���������ͻ��˱ң�Ϊ�Լ�֧�ֵ���Ӽ��ͣ������Ч������ͼ��ʾ��

![1701061](http://upload-images.jianshu.io/upload_images/1159224-7c7fa3ec3fee2dbc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

�Ҹ��˾���ͦ����ģ����Ծ����Լ�ʵ���������ť���ϻ�����˵���ȿ�ʵ�ֵ�Ч���ɣ�

![hoopview](http://upload-images.jianshu.io/upload_images/1159224-d2787b7f019e3765.gif?imageMogr2/auto-orient/strip)

���Ч����������popupwindow��࣬�����ǲ����Զ���view�ķ�ʽ��ʵ�֣�����˵˵���̡�

���ȴӻ��˵�Ч�����Կ���������������ťʱ������������֮�ϵģ���������Ҫ��FrameLayout���ʹ�ã�����������Ŀ�ȸ�����Ļ��С���߶ȸ���dpi�̶�������ʵ�ʳߴ�ʱ�����ģ�

![1701062](http://upload-images.jianshu.io/upload_images/1159224-e7203f8aab4ada5d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

�������view��ʼ���������ǿ������Է�Ϊ���飬����Բ��Բ�����֡�Բ�Ϸ����֣���������״̬�£�ֻ��Ҫ��onDraw�����л������������ݼ��ɡ����ڳ�ʼ�������н��Զ�������Ժͻ����Լ���ʼ������׼���ã�

```java
private void init(Context context, AttributeSet attrs) {
	//��ȡ�Զ�������
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
	// ���㱳����ı�ĳ��ȣ�Ĭ����������ť
	mChangeWidth = (int) (2 * mSmallRadius * 3 + 4 * margin);

}
```

��onMeasure�в��view�Ŀ�Ⱥ󣬸��ݿ�ȼ��������Բ��Բ�������һЩ��ص�����ֵ��

```java
@Override protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
	int widthSize = MeasureSpec.getSize(widthMeasureSpec);
	mWidth = getDefaultSize(widthSize, widthMeasureSpec);
	setMeasuredDimension(mWidth, mHeight);

	// ��ʱ�Ų����mWidthֵ���ټ���Բ�����꼰���ֵ
	cx = mWidth - mBigRadius;
	cy = mHeight - mBigRadius;
	// ��ԲԲ��
	circle = new PointF(cx, cy);
	// ������ť��Բ��
	circleOne = new PointF(cx - mBigRadius - mSmallRadius - margin, cy);
	circleTwo = new PointF(cx - mBigRadius - 3 * mSmallRadius - 2 * margin, cy);
	circleThree = new PointF(cx - mBigRadius - 5 * mSmallRadius - 3 * margin, cy);
	// ��ʼ�ı�����ı߽缴Ϊ��Բ���ĸ��߽��
	top = cy - mBigRadius;
	bottom = cy + mBigRadius;
}
```

��Ϊ�������漰�������ťչ���������Ĺ��̣������Ҷ��������¼���״̬��ֻ�����ض���״̬�²��ܽ���ĳЩ������

```java
private int mState = STATE_NORMAL;//��ǰչ��������״̬
private boolean mIsRun = false;//�Ƿ�����չ��������

//����״̬
public static final int STATE_NORMAL = 0;
//��ťչ��
public static final int STATE_EXPAND = 1;
//��ť����
public static final int STATE_SHRINK = 2;
//����չ��
public static final int STATE_EXPANDING = 3;
//��������
public static final int STATE_SHRINKING = 4;
```

��������ִ��onDraw�����ˣ��ȿ������룺

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

Բ�Ϸ������ֺ�Բ�ڵ�����������������һֱ���ڵģ������ҽ���������������`switch`֮�⣬����״̬�»���Բ��֮ǰ���������֣����չ��ʱ���Ʊ�����չ�����̺����֣�չ��״̬���ٴε�������������̺����֣���Ȼ�ڻ��Ʊ�����ķ�����Ҳ��Ҫ���ϻ��ƴ�Բ����ԲҲ��һֱ���ڵġ�

����Ļ��Ʒ�����

```java
/**
 * ��������Բ
 * @param canvas
 */
private void drawCircle(Canvas canvas) {
	left = cx - mBigRadius;
	right = cx + mBigRadius;
	canvas.drawCircle(cx, cy, mBigRadius, mBgPaint);
}


/**
 * ����Բ�����ʾ�����������
 * @param canvas
 */
private void drawCountText(Canvas canvas) {
	canvas.translate(0, -countMargin);
	//�������ֵĿ��
	float textWidth = mCountTextPaint.measureText(mCount, 0, mCount.length());
	canvas.drawText(mCount, 0, mCount.length(), (2 * mBigRadius - textWidth - 35) / 2, 0.2f, mCountTextPaint);
}


/**
 * ����Բ�ڵ�����
 * @param canvas
 */
private void drawCircleText(Canvas canvas) {
	StaticLayout layout = new StaticLayout(mText, mTextPaint, (int) (mBigRadius * Math.sqrt(2)), Layout.Alignment.ALIGN_CENTER, 1.0f, 0.0f, true);
	canvas.translate(mWidth - mBigRadius * 1.707f, mHeight - mBigRadius * 1.707f);
	layout.draw(canvas);
	canvas.save();
}


/**
 * ��������չ��������
 * @param canvas
 */
private void drawBackground(Canvas canvas) {
	left = cx - mBigRadius - mChange;
	right = cx + mBigRadius;
	canvas.drawRoundRect(left, top, right, bottom, mBigRadius, mBigRadius, mPopPaint);
	if ((mChange > 0) && (mChange <= 2 * mSmallRadius + margin)) {
		// ���Ƶ�һ����ť
		canvas.drawCircle(cx - mChange, cy, mSmallRadius, mBgPaint);
		// ���Ƶ�һ����ť�ڵ�����
		canvas.drawText(mDatas[0], cx - (mBigRadius - mSmallRadius) - mChange, cy + 15, mTextPaint);
	} else if ((mChange > 2 * mSmallRadius + margin) && (mChange <= 4 * mSmallRadius + 2 * margin)) {
		// ���Ƶ�һ����ť
		canvas.drawCircle(cx - mBigRadius - mSmallRadius - margin, cy, mSmallRadius, mBgPaint);
		// ���Ƶ�һ����ť�ڵ�����
		canvas.drawText(mDatas[0], cx - mBigRadius - mSmallRadius - margin - 20, cy + 15, mTextPaint);

		// ���Ƶڶ�����ť
		canvas.drawCircle(cx - mChange, cy, mSmallRadius, mBgPaint);
		// ���Ƶڶ�����ť�ڵ�����
		canvas.drawText(mDatas[1], cx - mChange - 20, cy + 15, mTextPaint);
	} else if ((mChange > 4 * mSmallRadius + 2 * margin) && (mChange <= 6 * mSmallRadius + 3 * margin)) {
		// ���Ƶ�һ����ť
		canvas.drawCircle(cx - mBigRadius - mSmallRadius - margin, cy, mSmallRadius, mBgPaint);
		// ���Ƶ�һ����ť�ڵ�����
		canvas.drawText(mDatas[0], cx - mBigRadius - mSmallRadius - margin - 16, cy + 15, mTextPaint);

		// ���Ƶڶ�����ť
		canvas.drawCircle(cx - mBigRadius - 3 * mSmallRadius - 2 * margin, cy, mSmallRadius, mBgPaint);
		// ���Ƶڶ�����ť�ڵ�����
		canvas.drawText(mDatas[1], cx - mBigRadius - 3 * mSmallRadius - 2 * margin - 25, cy + 15, mTextPaint);

		// ���Ƶ�������ť
		canvas.drawCircle(cx - mChange, cy, mSmallRadius, mBgPaint);
		// ���Ƶ�������ť�ڵ�����
		canvas.drawText(mDatas[2], cx - mChange - 34, cy + 15, mTextPaint);
	} else  if (mChange > 6 * mSmallRadius + 3 * margin) {
		// ���Ƶ�һ����ť
		canvas.drawCircle(cx - mBigRadius - mSmallRadius - margin, cy, mSmallRadius, mBgPaint);
		// ���Ƶ�һ����ť�ڵ�����
		canvas.drawText(mDatas[0], cx - mBigRadius - mSmallRadius - margin - 16, cy + 15, mTextPaint);

		// ���Ƶڶ�����ť
		canvas.drawCircle(cx - mBigRadius - 3 * mSmallRadius - 2 * margin, cy, mSmallRadius, mBgPaint);
		// ���Ƶڶ�����ť�ڵ�����
		canvas.drawText(mDatas[1], cx - mBigRadius - 3 * mSmallRadius - 2 * margin - 25, cy + 15, mTextPaint);

		// ���Ƶ�������ť
		canvas.drawCircle(cx - mBigRadius - 5 * mSmallRadius - 3 * margin, cy, mSmallRadius, mBgPaint);
		// ���Ƶ�������ť�ڵ�����
		canvas.drawText(mDatas[2], cx - mBigRadius - 5 * mSmallRadius - 3 * margin - 34, cy + 15, mTextPaint);
	}
	drawCircle(canvas);

}
```

Ȼ���ǵ���¼��Ĵ���ֻ�д������ڴ�Բ��ʱ�Żᴥ��չ���������Ĳ��������СԲʱ�ṩ��һ���ӿڸ��ⲿ���á�

```java
@Override public boolean onTouchEvent(MotionEvent event) {
	int action = event.getAction();
	switch (action) {
		case MotionEvent.ACTION_DOWN:
			//��������ʱ�򶯻��ڽ��У�������
			if (mIsRun) return true;
			PointF pointF = new PointF(event.getX(), event.getY());
			if (isPointInCircle(pointF, circle, mBigRadius)) { //����������ڴ�Բ�ڣ����ݵ������򵯳�����������ť
				if ((mState == STATE_SHRINK || mState == STATE_NORMAL) && !mIsRun) {
					//չ��
					mIsRun = true;//���Ǳ���������true����ΪonAnimationStart��onAnimationUpdate֮��ŵ���
					showPopMenu();
				} else {
					//����
					mIsRun = true;
					hidePopMenu();
				}
			} else { //�����㲻�ڴ�Բ��
				if (mState == STATE_EXPAND) { //�����չ��״̬
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

չ���������Ķ����Ǹı䱳����Ŀ�����ԵĶ�����������������Զ������ڿ��ֵ�ı�Ĺ�����ȥ���»�������view����Ϊһ��ʼ�Ҿ�ȷ���˴�ԲСԲ�İ뾶��СԲ�뱳����֮��ļ�࣬���Գ�ʼ��ʱ�Ѿ�������˱�����Ŀ�ȣ�

```java
mChangeWidth = (int) (2 * mSmallRadius * 3 + 4 * margin);
```

```java
/**
 * ����������
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
				//��������������״̬Ϊչ��
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
 * ���ص�����
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
				//��������������״̬Ϊ����
				mState = STATE_SHRINK;
			}
		});
		animator.setDuration(500);
		animator.start();
	}
}
```

������̿������ǵ�����������ʵ���Ͽ��ֵÿ�ı�һ�㣬�ͽ����е�����ػ�һ�Σ�ֻ�����ֺʹ�Բ�����ݵĳߴ缰λ�ö�û�б仯��ֻ�б�����Ŀ��ֵ�ڱ䣬���Բ�������Ч����

��xml�е�ʹ�ã�

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
		app:text="֧�ֻ��"
		app:count="1358"
		app:theme_color="#31A129"/>

	<com.xx.hoopcustomview.HoopView
		android:id="@+id/hoopview2"
		android:layout_width="match_parent"
		android:layout_height="wrap_content"
		android:layout_marginRight="10dp"
		app:text="�Ȼ��޵�"
		app:count="251"
		app:theme_color="#F49C11"/>
</LinearLayout>
```

activity��ʹ�ã�

```java
hoopview1 = (HoopView) findViewById(R.id.hoopview1);
hoopview1.setOnClickButtonListener(new HoopView.OnClickButtonListener() {
	@Override public void clickButton(View view, int num) {
		Toast.makeText(MainActivity.this, "hoopview1������" + num, Toast.LENGTH_SHORT).show();
	}
});
```


����ʵ�ֹ��̾�����������ԭʼЧ�������е�������������кܶ�覴ã��������ֵ�λ�þ������⣬����������ʱ��СԲ�ڵ����ֵ���ת������û��ʵ�֡�

�����ַ��
[https://github.com/shenhuniurou/HoopCustomView](https://github.com/shenhuniurou/HoopCustomView)