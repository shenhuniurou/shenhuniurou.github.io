---
layout: post
title: Material Design��ȫ�µĶ���
category: ��������
tags: Android����
---



[�ĵ���ַ](https://developer.android.com/training/material/animations.html#Reveal)

Material Design�еĶ�����Ϊ�û��ṩ�������������û�������Ӧ�ý��л���ʱ�ṩ�Ӿ������ԡ� Material Design��Ϊ��ť�������Ϊת���ṩһЩĬ�϶������� Android 5.0��API Level 21�������߰汾������������Щ������ͬʱҲ�ɴ����¶�����

### һ��������������

Ч��ͼ��

<img src="http://offfjcibp.bkt.clouddn.com/ripple.gif" width="30%" />

Material Design�Ĵ������������û��� UI Ԫ�ػ���ʱ���ڽӴ������ṩ��ʱ�Ӿ�ȷ�ϡ� �����ڰ�ť��Ĭ�ϴ�������ʹ��ȫ��?[RippleDrawable](https://developer.android.com/reference/android/graphics/drawable/RippleDrawable.html)����Բ���Ч��ʵ�ֲ�ͬ״̬���ת����
�ڴ��������£�Ӧ�����з�ʽָ����ͼ��������������ͼ XML ��Ӧ�ô˹��ܣ�

- ?android:attr/selectableItemBackground ָ���н�Ĳ��ơ�
- ?android:attr/selectableItemBackgroundBorderless ָ��Խ����ͼ�߽�Ĳ��ơ� ������һ���ǿձ�������ͼ��������������ƺ��趨�߽硣

�κ�view����**�ɵ��״̬**��������ʹ��RippleDrawable���ﵽˮ������Ч�����ұ��봦�ڿɵ��״̬���Ż���ֲ��ƶ���Ч����

�ڴ����п����������ã�

```java
RippleDrawableColorStateList stateList = getResources().getColorStateList(R.color.tint_state_color);
RippleDrawable rippleDrawable = new RippleDrawable(stateList, null, null);
view.setBackground(rippleDrawable);
```

> **ע�⣺**selectableItemBackgroundBorderless�� API Level 21 ���Ƴ��������ԡ�

���⣬��������?rippleԪ�ؽ�?[RippleDrawable](https://developer.android.com/reference/android/graphics/drawable/RippleDrawable.html)����Ϊһ�� XML ��Դ��
������Ϊ?[RippleDrawable](https://developer.android.com/reference/android/graphics/drawable/RippleDrawable.html)����ָ��һ����ɫ�����Ҫ�ı�Ĭ�ϴ���������ɫ����ʹ�������?android:colorControlHighlight���ԡ�
���Ҫ�˽������Ϣ�������?[RippleDrawable](https://developer.android.com/reference/android/graphics/drawable/RippleDrawable.html)���� API �ο��ĵ���

����������ϵͳ�Դ��Ĵ���������������ôʵ�ֵģ�Ϊʲôֻ��Ҫ��view��`background`����`foreground`�������ó�`?android:attr/selectableItemBackground`����`?android:attr/selectableItemBackgroundBorderless`�Ϳ���ʵ�ֲ��ƶ�����Ч�������������Ե��ȥ�����Կ�����·��`sdk/platforms/android-xx/data/res/values/attrs.xml`�ļ����ж�����ô�������ԣ�

```xml
<!-- Background drawable for bordered standalone items that need focus/pressed states. -->
<attr name="selectableItemBackground" format="reference" />
<!-- Background drawable for borderless standalone items that need focus/pressed states. -->
<attr name="selectableItemBackgroundBorderless" format="reference" />
```

�����뵽�����������Լ�Ȼ������app����Ч�ģ��ǿ��ܻ�����Theme�е����԰ɣ��Ǿ�ȥAndroidManifest�ļ��и����Themeһ��������ȥ�������`Base.V21.Theme.AppCompat.Light`���style�п���ȷʵ����������item���ԣ�

```xml
<item name="selectableItemBackground">?android:attr/selectableItemBackground</item>
<item name="selectableItemBackgroundBorderless">?android:attr/selectableItemBackgroundBorderless</item>
```

�������ﻹ�ǵ��õ�ϵͳ�Ķ�������ԣ���������׷����`android:Theme.Material`��`android:Theme.Material.Light`�У����Կ�����

```xml
<item name="selectableItemBackground">@drawable/item_background_material</item>
<item name="selectableItemBackgroundBorderless">@drawable/item_background_borderless_material</item>
```

Ȼ��sdk·����platforms\\\\android-xx\\\\data\\\\res\\\\drawable�����ҵ���Щ��Դ�ļ�����ͼ��

![](http://offfjcibp.bkt.clouddn.com/ripple_theme.png)

item_background_material�������ǣ�

```xml
<ripple xmlns:android="http://schemas.android.com/apk/res/android"
    android:color="?attr/colorControlHighlight">
    <item android:id="@id/mask">
        <color android:color="@color/white" />
    </item>
</ripple>
```

item_background_borderless_material�������ǣ�

```xml
<ripple xmlns:android="http://schemas.android.com/apk/res/android"
    android:color="?attr/colorControlHighlight" />
```

ϵͳ����������rippleԪ�ؽ�?RippleDrawable����Ϊһ�� XML ��Դ����ͨ����View��Դ�����ڹ��췽������������ȡbackground���Եģ�

```java
public View(Context context, @Nullable AttributeSet attrs, int defStyleAttr, int defStyleRes) {
        this(context);

        final TypedArray a = context.obtainStyledAttributes(
                attrs, com.android.internal.R.styleable.View, defStyleAttr, defStyleRes);

        if (mDebugViewAttributes) {
            saveAttributeData(attrs, a);
        }

        Drawable background = null;

        switch (attr) {
	        case com.android.internal.R.styleable.View_background:
		        background = a.getDrawable(attr);
		        break;

        .
        .
        .
}
```

Ҳ����˵�����backgroundʵ���Ͼ���RippleDrawable�ࡣ�����Ǿ����������RippleDrawable�ڲ���������ô���İɡ�

���ȹٷ��ĵ���RippleDrawable������
Drawable that shows a ripple effect in response to state changes. The anchoring position of the ripple for a given state may be specified by calling?`setHotspot(float, float)`with the corresponding state attribute identifier.
ͨ����ʾ������Ч������Ӧ״̬�ĸı䣬���ڸ���״̬�Ĳ��Ƶ�ê��λ�ÿ���ͨ�����þ��ж�Ӧ��״̬���Ա�ʶ����`setHotspot��float��float��`��ָ����

RippleDrawable�̳���`LayerDrawable`����`LayerDrawable`�Ǽ̳�`Drawable`��RippleDrawable����Ϊ����ӦView��statechange���ǾͿ���Drawable���жԵ��ʱ��״̬����ɡ�

```java
public boolean setState(@NonNull final int[] stateSet) {
	if (!Arrays.equals(mStateSet, stateSet)) {
		mStateSet = stateSet;
		return onStateChange(stateSet);
	}
	return false;
}
```

��Drawable����״̬����ʱ�����״̬�����鴫��onStateChange��������RippleDrawable����д��onStateChange��

```java
@Override
protected boolean onStateChange(int[] stateSet) {
	final boolean changed = super.onStateChange(stateSet);

	boolean enabled = false;
	boolean pressed = false;
	boolean focused = false;
	boolean hovered = false;

	for (int state : stateSet) {
		if (state == R.attr.state_enabled) {
			enabled = true;
		} else if (state == R.attr.state_focused) {
			focused = true;
		} else if (state == R.attr.state_pressed) {
			pressed = true;
		} else if (state == R.attr.state_hovered) {
			hovered = true;
		}
	}

	setRippleActive(enabled && pressed);
	setBackgroundActive(hovered || focused || (enabled && pressed), focused || hovered);

	return changed;
}
```

����`setRippleActive`��`setBackgroundActive`����������Ӧ�ÿ��Բµ���ʲô��˼�ˣ����ſ���

```java
private void setRippleActive(boolean active) {
	if (mRippleActive != active) {
		mRippleActive = active;
		if (active) {
			tryRippleEnter();
		} else {
			tryRippleExit();
		}
	}
}
```

���Drawable��enable=true��pressd=trueʱ�������`tryRippleEnter`����

```java
/**
 * Attempts to start an enter animation for the active hotspot. Fails if
 * there are too many animating ripples.
 */
private void tryRippleEnter() {
	if (mExitingRipplesCount >= MAX_RIPPLES) {
		// This should never happen unless the user is tapping like a maniac
		// or there is a bug that's preventing ripples from being removed.
		return;
	}

	if (mRipple == null) {
		final float x;
		final float y;
		if (mHasPending) {
			mHasPending = false;
			x = mPendingX;
			y = mPendingY;
		} else {
			x = mHotspotBounds.exactCenterX();
			y = mHotspotBounds.exactCenterY();
		}

		final boolean isBounded = isBounded();
		mRipple = new RippleForeground(this, mHotspotBounds, x, y, isBounded, mForceSoftware);
	}

	mRipple.setup(mState.mMaxRadius, mDensity);
	mRipple.enter(false);
}
```

����������ǿ���֪��Ҫ��ʼ�����ƶ�����Ч���ˡ�mRipple ��RippleForeground���ʵ����Ȼ����û����RippleForeground�����ҵ�setup��enter����������RippleForeground�̳���RippleComponent�࣬���ǣ�����������з�����������������

```java
public final void setup(float maxRadius, int densityDpi) {
	if (maxRadius >= 0) {
		mHasMaxRadius = true;
		mTargetRadius = maxRadius;
	} else {
		mTargetRadius = getTargetRadius(mBounds);
	}

	mDensityScale = densityDpi * DisplayMetrics.DENSITY_DEFAULT_SCALE;

	onTargetRadiusChanged(mTargetRadius);
}
```

```java
/**
 * Starts a ripple enter animation.
 *
 * @param fast whether the ripple should enter quickly
 */
public final void enter(boolean fast) {
	cancel();

	mSoftwareAnimator = createSoftwareEnter(fast);

	if (mSoftwareAnimator != null) {
		mSoftwareAnimator.start();
	}
}
```

setup�ǳ�ʼ��һϵ�в�����enter����һ����������ʼ������

```java
@Override
protected Animator createSoftwareEnter(boolean fast) {
	// Bounded ripples don't have enter animations.
	if (mIsBounded) {
		return null;
	}

	final int duration = (int) (1000 * Math.sqrt(mTargetRadius / WAVE_TOUCH_DOWN_ACCELERATION * mDensityScale) + 0.5);

	final ObjectAnimator tweenRadius = ObjectAnimator.ofFloat(this, TWEEN_RADIUS, 1);
	tweenRadius.setAutoCancel(true);
	tweenRadius.setDuration(duration);
	tweenRadius.setInterpolator(LINEAR_INTERPOLATOR);
	tweenRadius.setStartDelay(RIPPLE_ENTER_DELAY);

	final ObjectAnimator tweenOrigin = ObjectAnimator.ofFloat(this, TWEEN_ORIGIN, 1);
	tweenOrigin.setAutoCancel(true);
	tweenOrigin.setDuration(duration);
	tweenOrigin.setInterpolator(LINEAR_INTERPOLATOR);
	tweenOrigin.setStartDelay(RIPPLE_ENTER_DELAY);

	final ObjectAnimator opacity = ObjectAnimator.ofFloat(this, OPACITY, 1);
	opacity.setAutoCancel(true);
	opacity.setDuration(OPACITY_ENTER_DURATION_FAST);
	opacity.setInterpolator(LINEAR_INTERPOLATOR);

	final AnimatorSet set = new AnimatorSet();
	set.play(tweenOrigin).with(tweenRadius).with(opacity);

	return set;
}
```

�����洴�������Ĵ�����Կ�����ʵ������һ����ϵ����Զ�����Ȼ���Զ������������Բ��ư뾶`TWEEN_RADIUS`���������ĵ�`TWEEN_ORIGIN`�Ͳ��ƵĲ�͸����`OPACITY`��ͨ�����������ԵĹ��ɱ仯�õ�һ�����ϵĶ��������Ͼ���ǰ�����ƶ���Ч����ʵ�ֹ��̡�


```java
private void setBackgroundActive(boolean active, boolean focused) {
	if (mBackgroundActive != active) {
		mBackgroundActive = active;
		if (active) {
			tryBackgroundEnter(focused);
		} else {
			tryBackgroundExit();
		}
	}
}
```

`mBackground`��RippleBackground���ʵ������RippleForeground��ͬ���ǣ���������ֻ�Ǹı��˲�͸���ȡ�

```java
@Override
protected Animator createSoftwareEnter(boolean fast) {
	// Linear enter based on current opacity.
	final int maxDuration = fast ? OPACITY_ENTER_DURATION_FAST : OPACITY_ENTER_DURATION;
	final int duration = (int) ((1 - mOpacity) * maxDuration);

	final ObjectAnimator opacity = ObjectAnimator.ofFloat(this, OPACITY, 1);
	opacity.setAutoCancel(true);
	opacity.setDuration(duration);
	opacity.setInterpolator(LINEAR_INTERPOLATOR);

	return opacity;
}
```

���Ϸ����Ķ�����ָ����viewʱ������enter���ƶ���������ָ̧��ʱstateҲ��ı䣬�����һ��exit����������Ͳ���ϸ�����ˡ�

### ����ʹ�ý�¶Ч��

Ч��ͼ��

<img src="http://offfjcibp.bkt.clouddn.com/circle.gif" width="30%" />

����Ҫ��ʾ������һ��UIԪ��ʱ����¶������Ϊ�û��ṩ�Ӿ������ԡ�[ViewAnimationUtils.createCircularReveal()](https://developer.android.com/reference/android/view/ViewAnimationUtils.html#createCircularReveal(android.view.View, int, int, float, float))�����ܹ�Ϊ�ü�������Ӷ����Խ�¶��������ͼ��

```java
/* @param view The View will be clipped to the animating circle.Ҫ���ػ���ʾ��view
 * @param centerX The x coordinate of the center of the animating circle, relative to <code>view</code>.������ʼ�����ĵ�X
 * @param centerY The y coordinate of the center of the animating circle, relative to <code>view</code>.������ʼ�����ĵ�Y
 * @param startRadius The starting radius of the animating circle.������ʼ�뾶
 * @param endRadius The ending radius of the animating circle.���������뾶
 */
public static Animator createCircularReveal(View view,
		int centerX,  int centerY, float startRadius, float endRadius) {
	return new RevealAnimator(view, centerX, centerY, startRadius, endRadius);
}
```
RevealAnimator��֮ǰ�Ķ���ʹ��ûʲô����ͬ���������ü������ͼ�������ʵ�ָ��ָ�������Ч���ö�����Ҫ�������ػ�����ʾһ��view���ı�view�Ĵ�С�ȹ���Ч����

��ʾview��

```java
final TextView tv9 = (TextView) findViewById(R.id.tv9);

findViewById(R.id.content_main).setOnClickListener(new View.OnClickListener() {
	@Override public void onClick(View v) {
		// get the center for the clipping circle
		int cx = (tv9.getRight() - tv9.getLeft()) / 2;
		int cy = (tv9.getBottom() - tv9.getTop()) / 2;

		// get the final radius for the clipping circle
		int finalRadius = Math.max(tv9.getWidth(), tv9.getHeight());

		// create the animator for this view (the start radius is zero)
		final Animator anim = ViewAnimationUtils.createCircularReveal(tv9, cx, cy, 0, finalRadius);

		tv9.setVisibility(View.VISIBLE);

		anim.start();
	}
});
```

����view��

```java
final TextView tv9 = (TextView) findViewById(R.id.tv9);

tv9.setOnClickListener(new View.OnClickListener() {
	@Override public void onClick(View v) {
		// get the center for the clipping circle
		int cx = (tv9.getRight() - tv9.getLeft()) / 2;
		int cy = (tv9.getBottom() - tv9.getTop()) / 2;

		// get the final radius for the clipping circle
		int initRadius = Math.max(tv9.getWidth(), tv9.getHeight());

		// create the animator for this view (the start radius is zero)
		final Animator anim = ViewAnimationUtils.createCircularReveal(tv9, cx, cy, initRadius, 0);

		anim.addListener(new AnimatorListenerAdapter() {
			@Override public void onAnimationEnd(Animator animation) {
				super.onAnimationEnd(animation);
				// make the view visible and start the animation
				tv9.setVisibility(View.INVISIBLE);
			}
		});
		anim.start();
	}
});
```

����������С��

```java
Animator animator = ViewAnimationUtils.createCircularReveal(view, view.getWidth() / 2, view.getHeight() / 2, view.getWidth(), 0);
animator.setInterpolator(new LinearInterpolator());
animator.setDuration(1000);
animator.start();
```

�����Ͻ���չ��

```java
Animator animator = ViewAnimationUtils.createCircularReveal(view,0,0,0,(float) Math.hypot(view.getWidth(), view.getHeight()));
animator.setDuration(1000);
animator.start();
```

### ����ʹ��ת������

Ч��ͼ�Թ���Ԫ�ص�ת������Ϊ����

<img src="http://offfjcibp.bkt.clouddn.com/share.gif" width="30%" />

MaterialDesignӦ���еĲ�����Ϊת��͸��ͨ��Ԫ��֮����ƶ���ת���ṩ��ͬ״̬֮����Ӿ����ӡ���Ϊ���롢�˳�ת���Լ�������Ϊ֮��Ĺ���Ԫ��ת��ָ�����ƶ�������5.0֮ǰ�����ǿ�����startActivity֮�����overridePendingTransition��ָ��Activity��ת��������

- **����**ת��������������Ϊ����ͼ��ν��볡�������磬��**�ֽ�**����ת���У���ͼ������Ļ����볡����������Ļ���ġ�
- **�˳�**ת��������������Ϊ��Ӧ����Ϊ����ʾ��ͼ����˳����������磬��**�ֽ�**�˳�ת���У���ͼ������Ļ�����˳�������
- **����Ԫ��**ת������������������Ϊת��֮�乲�����ͼ�������Щ������Ϊ��ת���� ���磬�������������Ϊӵ����ͬ��ͼ�񣬵���λ�����С��ͬ��**changeImageTransform**����Ԫ��ת��������Щ������Ϊ֮��ƽ����ת��������ͼ��

Android 5.0��API Level 21��֧����Щ�������˳�ת��������ͨ���ɶ�����

- *�ֽ�*?- �ӳ�������������Ƴ���ͼ��
- *����*?- �ӳ�����Ե������Ƴ���ͼ��
- *���뵭��*?- ͨ������͸�����ڳ�����������Ƴ���ͼ��

Ҳ֧����Щ����Ԫ��ת����������Ԫ�صĹ��ɶ�����

- *changeBounds*?- ΪĿ����ͼ�Ĵ�С��Ӷ�����
- *changeClipBounds*?- ΪĿ����ͼ�Ĳü���С��Ӷ�����
- *changeTransform*?- ΪĿ����ͼ�����š���ת��λ����Ӷ�����
- *changeImageTransform*?- ΪĿ��ͼƬ�����š���ת��λ����Ӷ�����

#### ָ��ת������

Ҫ��ʹ���µ�ת�����������Լ̳�Material Design�������style�����ָ����

```xml
<!-- ����ʹ��transitions -->
<item name="android:windowContentTransitions">true</item>
<!-- ָ�����롢�˳������ء����½���ʱ��transitions -->
<item name="android:windowEnterTransition">@transition/explode</item>
<item name="android:windowExitTransition">@transition/explode</item>
<item name="android:windowReturnTransition">@transition/explode</item>
<item name="android:windowReenterTransition">@transition/explode</item>
<!-- ָ�����롢�˳������ء����½���ʱ�Ĺ���transitions -->
<item name="android:windowSharedElementEnterTransition">@transition/change_image_transform</item>
<item name="android:windowSharedElementExitTransition">@transition/change_image_transform</item>
<item name="android:windowSharedElementReturnTransition">@transition/change_image_transform</item>
<item name="android:windowSharedElementReenterTransition">@transition/change_image_transform</item>
```

���У�change_image_transform�������£�

```xml
<!-- res/transition/change_image_transform.xml -->
<!-- (see also Shared Transitions below) -->
<transitionSet xmlns:android="http://schemas.android.com/apk/res/android">
  <changeImageTransform/>
</transitionSet>
```

���Ҫ�������п�����������ת������Ҫ����`Window.requestFeature()`������

```java
// ����ʹ��transitions
getWindow().requestFeature(Window.FEATURE_CONTENT_TRANSITIONS);

// ָ�����롢�˳������ء����½���ʱ��transitions
getWindow().setEnterTransition(new Explode());
getWindow().setExitTransition(new Explode());
getWindow().setReturnTransition(new Explode());
getWindow().setReenterTransition(new Explode());

// ָ�����롢�˳������ء����½���ʱ�Ĺ���transitions
getWindow().setSharedElementEnterTransition(new ChangeTransform());
getWindow().setSharedElementExitTransition(new ChangeTransform());
getWindow().setSharedElementReturnTransition(new ChangeTransform());
getWindow().setSharedElementReenterTransition(new ChangeTransform());
```

��ͨת��������

���м̳���visibility�඼������Ϊ���롢�˳��Ĺ��ȶ���������������Զ��������˳�ʱ�Ķ���Ч����ֻ��Ҫ�̳�Visibility������onAppear��onDisappear���������������˳��Ķ�����ϵͳ�ṩ������Ĭ�Ϸ�ʽ��

- explode ����Ļ����������Ƴ���ͼ
- slide ����Ļ��Ե������Ƴ���ͼ
- fade �ı���ͼ��͸����

����xml��ָ���Զ���Ľ��롢�˳��Ĺ��ȶ�����Ҫ�ȶԶ������ж��壺

```xml
<transition class="my.app.transition.CustomTransition"/>
```
?
> **ע��**������CustomTransition�������Զ���Ķ�����������̳���Visibility��

������ͨת�������ķ�ʽ����һ��Activity��������startActivity�����д���һ��ActivityOptions��Bundle����

```java
ActivityOptions options = ActivityOptions.makeSceneTransitionAnimation(activity); 
startActivity(intent, options.toBundle());
```

������÷���Ҳ�߱�ת��Ч������ô�ڷ��ص�Activity�в�Ҫ�ٵ���finish����������Ӧ��ʹfinishAfterTransition������һ��Activity���ú�����ȴ�����ִ����ϲŽ�����Activity��

����ת��������

���Ҫ���������й���Ԫ�ص�Activity֮��ʹ��ת����������ô��

- 1�����������ô�������ת����android:windowContentTransitions
- 2����Theme��ָ��һ������Ԫ��ת����
- 3����transitions����Ϊxml��Դ��
- 4������?android:transitionName���Զ����������еĹ���Ԫ��ָ��һ��ͨ�����ơ�
- 5��ʹ��?`ActivityOptions.makeSceneTransitionAnimation()`������

```java
// get the element that receives the click event
final View imgContainerView = findViewById(R.id.img_container);

// get the common element for the transition in this activity
final View androidRobotView = findViewById(R.id.image_small);

// define a click listener
imgContainerView.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View view) {
        Intent intent = new Intent(this, Activity2.class);
        // create the transition animation - the images in the layouts
        // of both activities are defined with android:transitionName="robot"
        ActivityOptions options = ActivityOptions
            .makeSceneTransitionAnimation(this, androidRobotView, "robot");
        // start the new activity
        startActivity(intent, options.toBundle());
    }
});
```

���Ҫ�ڴ��������ɹ���view����ô��Ҫ����`View.setTransitionName()`���������������еĹ���Ԫ��ָ��һ��ͨ�����ơ�
����ж������Ԫ�أ������ͨ��Pair���а�װ����

```java
ActivityOptions options = ActivityOptions.makeSceneTransitionAnimation(activity,
      Pair.create(view1, "name1"),//����view1��view2�����TextView����ImageView�ȣ���Ҫת��View���Ͳſ���
      Pair.create(view2, "name2"));      
startActivity(intent,.toBundle());
```

����ʱ�����Ҫ�߱�ת����������ôҲ��Ҫ��finish�������finishAfterTransition������һ��Activity��


### ʹ�������˶�

��Ϊ�����˶������Զ����Լ�������������Щ����������һ������׼��������ó�������д������Ͳ���˵�ˡ�

### ��ͼ״̬�ı�

Android 5.0��ԭ�е�ͼƬѡ��������ɫѡ�����Ͻ�������ǿ�������ǿؼ��ܸ��ݲ�ͬ��״̬��ʾ��ͬ�ı���ͼƬ������������״̬�л�ʱָ��һ�������������ӹ���Ч���������û�������ͻ���ص����ݡ�

StateListAnimator���ͼƬѡ��������ɫѡ�������ƣ����Ը���view��״̬�ı���ֲ�ͬ�Ķ���Ч����ͨ��xml���ǿ��Թ�����Ӧ��ͬ״̬�Ķ����ϼ�����ʹ�÷�ʽҲ�ǳ��򵥣��ڶ�Ӧ��״ָ̬��һ�����Զ������ɣ�

```xml
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:state_pressed="true">
        <set>
            <objectAnimator android:propertyName="translationZ"
                            android:duration="200"
                            android:valueTo="20dp"
                            android:valueType="floatType"/>
        </set>
    </item>
    <item android:state_enabled="true" android:state_pressed="false">
        <set>
            <objectAnimator android:propertyName="translationZ"
                            android:duration="200"
                            android:valueTo="0"
                            android:valueType="floatType"/>
        </set>
    </item>
</selector>
```

�������������ؼ��ɣ�

```java
TextView tv11 = (TextView) findViewById(R.id.tv11);
StateListAnimator stateLAnim = AnimatorInflater.loadStateListAnimator(this,R.drawable.selector_for_button);
tv11.setStateListAnimator(stateLAnim);
```

�̳���Material����󣬰�ťĬ��ӵ����z���Զ����������ȡ������Ĭ��״̬�����԰�״̬����ָ��Ϊnull��

����StateListAnimator��ָ��״̬�л������Զ����⣬������ͨ��AnimatedStateListDrawable��ָ��״̬�л���֡������

```xml
<animated-selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:id="@+id/pressed" android:drawable="@drawable/btn_check_15" android:state_pressed="true"/>
    <item android:id="@+id/normal"  android:drawable="@drawable/btn_check_0"/>
    <transition android:fromId="@+id/normal" android:toId="@+id/pressed">
        <animation-list>
            <item android:duration="20" android:drawable="@drawable/btn_check_0"/>
            <item android:duration="20" android:drawable="@drawable/btn_check_1"/>
            <item android:duration="20" android:drawable="@drawable/btn_check_2"/>
        </animation-list>
    </transition>
</animated-selector>
```

֡��������Դ�ļ�ֱ����xml����Ϊview��background���ɡ�

### ʸ��ͼ����

Ч��ͼ:
<img src="http://offfjcibp.bkt.clouddn.com/vector.gif" width="30%" />

����drawable�ж���һ��ʸ��ͼ��

```xml
<?xml version="1.0" encoding="utf-8"?>
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:height="200dp"
    android:width="200dp"
    android:viewportHeight="400"
    android:viewportWidth="400">
    ����
    <group
        android:name="rotationGroup"
        android:pivotX="0"
        android:pivotY="0">
        ������
        <path
            android:name="star"
            android:pathData="M 100,100 h 200 l -200 150 100 -250 100 250 z"
            android:strokeColor="@color/colorPrimary"
            android:strokeLineCap="round"
            android:strokeWidth="10"/>
        ����
    </group>
</vector>
```

Ȼ����anim�ж��嶯����

```xml
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <objectAnimator
        android:propertyName="trimPathStart"
        android:valueFrom="0"
        android:valueTo="1"
        android:valueType="floatType"
        android:duration="2000"
        android:repeatMode="reverse"
        android:repeatCount="-1"
        android:interpolator="@android:interpolator/accelerate_decelerate"/>
</set>
```

�����drawable�ж���һ��`animated-vector`����������Դָ����drawable����ֵ��ʸ��ͼ��

```xml
<?xml version="1.0" encoding="utf-8"?>
<animated-vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:drawable="@drawable/vector_drawable">
    <target
        android:name="star"
        android:animation="@anim/animation"/>
   
</animated-vector>
```

> **ע��**������drawable����ֵ��ǰ�����Ƕ����ʸ��ͼ��target��nameҪ��ʸ��ͼ��path��nameһ����animation����ǰ�涨��Ķ�����Դ�ļ���

��view��xml��ʹ���Լ��ڴ����п�ʼ������

```xml
<ImageView
	android:id="@+id/iv"
	android:layout_width="wrap_content"
	android:layout_height="wrap_content"
	android:layout_margin="20dp"
	app:srcCompat="@drawable/anim_vector_drawable"
	android:layout_gravity="center"/>
```

```java
ImageView iv = (ImageView) findViewById(R.id.iv);
Drawable drawable = iv.getDrawable();
if (drawable instanceof Animatable) {
	((Animatable) drawable).start();
}
```

��������ʾ��[Demo��ַ](https://github.com/shenhuniurou/BlogDemos/tree/master/LollipopAnimation)

### �ο��ĵ�
- [���嶨�ƶ���](https://developer.android.com/training/material/animations.html#Transitions)
- [AndroidMaterialDesign����֮RippleDrawable](http://blog.csdn.net/huyuchaoheaven/article/details/47103613)
- [Android5.0�����ԡ���ȫ�µĶ�����animation��](http://www.cnblogs.com/McCa/p/4465574.html)
- [��svgʸ��ͼʵ�ֶ���Ч��](http://www.voidcn.com/blog/qq_17583407/article/p-5928494.html)