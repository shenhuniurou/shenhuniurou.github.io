---
layout: post
title: support design library中一些组件的使用
category: 技术分享
tags: support design library
---


## 概述

最近利用上下班在路上的时间一直在看一些技术文章，目的当然是学习别人的开发经验来提高自己的技术水平。读别人的文章很有趣，即使是我以为很简单的知识点，别人也能写出一些新花样来，这时你会惊奇的发现，原来这玩意还能这么玩！！谷歌推出`android.support.design`中的组件也好久了，写demo的人也很多，但是在实际开发过程中使用率还没有达到很普遍的程度，一是因为开发者对这些新的东西还没有比较深的学习和认识，我相信即使是现在仍然有很多人还不知道怎么使用或者不清楚他们的实现原理；二是因为开发习惯，比如很多人对`ListView`用的比较熟悉，所以当他有一个需求需要用到列表显示数据时，他当然首选`ListView`而不是`RecyclerView`了，这也是由于上一点原因，对新组件的不熟悉导致不愿意去在实际开发中使用这些东西。但，现在是时候放弃那些老古董了。

`android.support.design`中的组件我最常用的也就是`AppBarLayout`和`TabLayout`了，因为它很好的取代了**[ViewPagerIndicator](https://github.com/JakeWharton/ViewPagerIndicator)**这个开源组件，而其他的比如`FloatingActionButton`、`NavigationView`、`Snackbar`、`TextInputLayout`、`CollapsingToolbarLayout`这些都使用的很少，虽然不常用到部分原因是产品设计出的UI原型可能不会遵循`Material Design`，但是掌握他们的基本使用方法是有必要的，万一哪天就用上了呢。

## NavigationView

`NavigationView`一般和v4包中的`DrawerLayout`结合使用，虽然目前大部分的app放弃了这种侧滑显示菜单的设计方式而改成顶部或底部Tab切换菜单，但仍有一些使用到的。它本质上是一个`FrameLayout`，当在`Android Studio`新建工程到选择`Activity`模式时选择`Navigation Drawer Activity`时会自动帮你构建一个带有侧滑抽屉效果的`Activity`，它默认的`xml`是这样的：

```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.v4.widget.DrawerLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/drawer_layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fitsSystemWindows="true"
    tools:openDrawer="start">

    <include
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        layout="@layout/app_bar_main"/>

    <android.support.design.widget.NavigationView
        android:id="@+id/nav_view"
        android:layout_width="wrap_content"
        android:layout_height="match_parent"
        android:layout_gravity="start"
        android:fitsSystemWindows="true"
        app:headerLayout="@layout/nav_header_main"
        app:menu="@menu/activity_main_drawer"/>

</android.support.v4.widget.DrawerLayout>
```

它是将`app_bar_main`这个`view`和`NavigationView`同时包裹在`DrawerLayout`中，而`app_bar_main`也就是相当于主屏幕显示内容的区域，它的`xml`就和我们平常建工程的主页面是一样的：

```
<?xml version="1.0" encoding="utf-8"?>
<android.support.design.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fitsSystemWindows="true"
    tools:context="com.xx.design.MainActivity">

    <android.support.design.widget.AppBarLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:theme="@style/AppTheme.AppBarOverlay">

        <android.support.v7.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="?attr/actionBarSize"
            android:background="?attr/colorPrimary"
            app:popupTheme="@style/AppTheme.PopupOverlay"/>

    </android.support.design.widget.AppBarLayout>

    <include layout="@layout/content_main"/>

    <android.support.design.widget.FloatingActionButton
        android:id="@+id/fab"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="bottom|end"
        android:layout_margin="@dimen/fab_margin"
        app:srcCompat="@android:drawable/ic_dialog_email"/>

</android.support.design.widget.CoordinatorLayout>
```

在来看看`NavigationView`，我们只需要看三个属性：

- `layout_gravity`：这个属性控制侧滑菜单的位置是左边还是右边。它有`start`、`end`、`left`、`right`四个值，其中`start`和`left`都表示位置在左边，`end`和`right`都表示位置在右边，但官方推荐使用`start`和`end`，因为使用另外两个可能会在滑动过程中导致一些问题出现。
![1701111](http://upload-images.jianshu.io/upload_images/1159224-c33e37c7b0590cc9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> `DrawerLayout`中的`openDrawer`属性是控制`NavigationView`的打开和关闭的方向，所以它的值一般和`layout_gravity`设置成一样。

- `app:headerLayout`：它是侧滑页面的顶部，一般放用户头像、性别、个性签名这些view。默认的`nav_header_main.xml`：

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="@dimen/nav_header_height"
    android:background="@drawable/side_nav_bar"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    android:theme="@style/ThemeOverlay.AppCompat.Dark"
    android:orientation="vertical"
    android:gravity="bottom">

    <ImageView
        android:id="@+id/imageView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:paddingTop="@dimen/nav_header_vertical_spacing"
        app:srcCompat="@android:drawable/sym_def_app_icon"/>

    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:paddingTop="@dimen/nav_header_vertical_spacing"
        android:text="Android Studio"
        android:textAppearance="@style/TextAppearance.AppCompat.Body1"/>

    <TextView
        android:id="@+id/textView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="android.studio@android.com"/>

</LinearLayout>

```

- `app:menu`：是一些菜单按钮的xml，默认的`activity_main_drawer.xml`：

```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">

    <group android:checkableBehavior="single">
        <item
            android:id="@+id/nav_camera"
            android:icon="@drawable/ic_menu_camera"
            android:title="Import"/>
        <item
            android:id="@+id/nav_gallery"
            android:icon="@drawable/ic_menu_gallery"
            android:title="Gallery"/>
        <item
            android:id="@+id/nav_slideshow"
            android:icon="@drawable/ic_menu_slideshow"
            android:title="Slideshow"/>
        <item
            android:id="@+id/nav_manage"
            android:icon="@drawable/ic_menu_manage"
            android:title="Tools"/>
    </group>

    <item android:title="Communicate">
        <menu>
            <item
                android:id="@+id/nav_share"
                android:icon="@drawable/ic_menu_share"
                android:title="Share"/>
            <item
                android:id="@+id/nav_send"
                android:icon="@drawable/ic_menu_send"
                android:title="Send"/>
        </menu>
    </item>
    
</menu>
```

但是这两个属性不是必须有的，一张图看清`NavigationView`、`headerLayout`和`menu`的关系：


![1701112](http://upload-images.jianshu.io/upload_images/1159224-f407efb0b9c66bc6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

页面布局就是这样了，在`Activity`中初始化`DrawerLayout `和`NavigationView `并设置监听：

```java
DrawerLayout drawer = (DrawerLayout) findViewById(R.id.drawer_layout);
ActionBarDrawerToggle toggle = new ActionBarDrawerToggle(this, drawer, toolbar, R.string.navigation_drawer_open, R.string.navigation_drawer_close);
// 监听侧滑开关的切换
drawer.addDrawerListener(toggle);
toggle.syncState();

NavigationView navigationView = (NavigationView) findViewById(R.id.nav_view);
navigationView.setNavigationItemSelectedListener(this);
```

还需要实现`NavigationView.OnNavigationItemSelectedListener`这个接口，是点击`menu`中的菜单的监听，并且在`onNavigationItemSelected`方法中处理回调：

```java
public boolean onNavigationItemSelected(MenuItem item) {
	// Handle navigation view item clicks here.
	int id = item.getItemId();

	if (id == R.id.nav_camera) {
		// Handle the camera action
	} else if (id == R.id.nav_gallery) {

	} else if (id == R.id.nav_slideshow) {

	} else if (id == R.id.nav_manage) {

	} else if (id == R.id.nav_share) {

	} else if (id == R.id.nav_send) {

	}

	DrawerLayout drawer = (DrawerLayout) findViewById(R.id.drawer_layout);
	drawer.closeDrawer(GravityCompat.START);
	return true;
}
```

退出界面时需要关闭菜单：

```java
@Override public void onBackPressed() {
	DrawerLayout drawer = (DrawerLayout) findViewById(R.id.drawer_layout);
	if (drawer.isDrawerOpen(GravityCompat.START)) {
		drawer.closeDrawer(GravityCompat.START);
	} else if (drawer.isDrawerOpen(GravityCompat.END)) {
		drawer.closeDrawer(GravityCompat.END);
	} else {
		super.onBackPressed();
	}
}
```

这么看来，使用还是挺简单的，开发者仅仅需要关注布局和点击事件的处理，界面的交互google已经帮我们都处理好了。

## FloatingActionButton

`FloatingActionButton`实质上是`ImageButton`，它的使用也很简单：

```java
FloatingActionButton fab = (FloatingActionButton) findViewById(R.id.fab);
fab.setOnClickListener(new View.OnClickListener() {
	@Override public void onClick(View view) {
		//TODO: do sth
	}
});
```

目前一般遵循MD设计风格的app都会使用到`FAB`，它一般放在屏幕右下角，比如这样：

![smooth.png](http://upload-images.jianshu.io/upload_images/1159224-e9af92fa280982f9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

它的作用比较简单，即提供一个点击事件，去做相应的操作，现在我们要研究的是滑动屏幕时`FAB`的显示和隐藏。

常见的隐藏和显示效果有两种：一种是缩放动画，一种是平移动画，当`FAB`与`CoordinatorLayout`一起使用时，给它设置`app:layout_behavior`属性：

```xml
<android.support.design.widget.FloatingActionButton
	android:id="@+id/fab"
	android:layout_width="wrap_content"
	android:layout_height="wrap_content"
	android:layout_gravity="bottom|end"
	android:layout_margin="@dimen/fab_margin"
	app:layout_behavior="@string/fab_custom_behavior"
	app:srcCompat="@android:drawable/ic_dialog_email"/>
```

```xml
<string name="fab_custom_behavior">com.xx.design.ScrollAwareFABBehavior</string>
```

`ScrollAwareFABBehavior`是继承默认的`FloatingActionButton.Behavior`，而`CoordinatorLayout`又实现了`NestedScrollingParent`接口来监听列表的滚动，所以要隐藏或显示`FAB`，只需要`onStartNestedScroll`方法中选择垂直方向`ViewCompat.SCROLL_AXIS_VERTICAL`上的滚动，在`onNestedScroll`方法中监听滑动是向上还是向下。另外谷歌官方在**22.2.1**版本给`FAB`加了`hide`和`show`两个方法，它的效果是做缩放动画，**22.2.1**之前的版本需要自己实现，可以这么写：

```java
@Override
public void onNestedScroll(final CoordinatorLayout coordinatorLayout, final FloatingActionButton child, final View target, final int dxConsumed, final int dyConsumed, final int dxUnconsumed, final int dyUnconsumed) {
	super.onNestedScroll(coordinatorLayout, child, target, dxConsumed, dyConsumed, dxUnconsumed,
		dyUnconsumed);
	if (dyConsumed > 0 && !this.mIsAnimatingOut && child.getVisibility() == View.VISIBLE) {
		// User scrolled down and the FAB is currently visible -> hide the FAB
		animateOut(child);
        // child.hide();
	} else if (dyConsumed < 0 && child.getVisibility() != View.VISIBLE) {
		// User scrolled up and the FAB is currently not visible -> show the FAB
		animateIn(child);
        // child.show();
	}
}

// Same animation that FloatingActionButton.Behavior uses to hide the FAB when the AppBarLayout exits
private void animateOut(final FloatingActionButton button) {
	if (Build.VERSION.SDK_INT >= 14) {
		ViewCompat.animate(button)
			.scaleX(0.0F)
			.scaleY(0.0F)
			.alpha(0.0F)
			.setInterpolator(INTERPOLATOR)
			.withLayer()
			.setListener(new ViewPropertyAnimatorListener() {
				public void onAnimationStart(View view) {
					ScrollAwareFABBehavior.this.mIsAnimatingOut = true;
				}


				public void onAnimationCancel(View view) {
					ScrollAwareFABBehavior.this.mIsAnimatingOut = false;
				}


				public void onAnimationEnd(View view) {
					ScrollAwareFABBehavior.this.mIsAnimatingOut = false;
					view.setVisibility(View.GONE);
				}
			})
			.start();
	} else {
		Animation anim = AnimationUtils.loadAnimation(button.getContext(),
			android.support.design.R.anim.design_fab_out);
		anim.setInterpolator(INTERPOLATOR);
		anim.setDuration(200L);
		anim.setAnimationListener(new Animation.AnimationListener() {
			public void onAnimationStart(Animation animation) {
				ScrollAwareFABBehavior.this.mIsAnimatingOut = true;
			}


			public void onAnimationEnd(Animation animation) {
				ScrollAwareFABBehavior.this.mIsAnimatingOut = false;
				button.setVisibility(View.GONE);
			}


			@Override public void onAnimationRepeat(final Animation animation) {
			}
		});
		button.startAnimation(anim);
	}
}

// Same animation that FloatingActionButton.Behavior uses to show the FAB when the AppBarLayout enters
private void animateIn(FloatingActionButton button) {
	button.setVisibility(View.VISIBLE);
	if (Build.VERSION.SDK_INT >= 14) {
		ViewCompat.animate(button)
			.scaleX(1.0F)
			.scaleY(1.0F)
			.alpha(1.0F)
			.setInterpolator(INTERPOLATOR)
			.withLayer()
			.setListener(null)
			.start();
	} else {
		Animation anim = AnimationUtils.loadAnimation(button.getContext(),
			android.support.design.R.anim.design_fab_in);
		anim.setDuration(200L);
		anim.setInterpolator(INTERPOLATOR);
		button.startAnimation(anim);
	}
}
```

竖直方向上的平移动画：

```java
@Override
public void onNestedScroll(final CoordinatorLayout coordinatorLayout, final FloatingActionButton child, final View target, final int dxConsumed, final int dyConsumed, final int dxUnconsumed, final int dyUnconsumed) {
	super.onNestedScroll(coordinatorLayout, child, target, dxConsumed, dyConsumed, dxUnconsumed,
		dyUnconsumed);
	if (dyConsumed > 0 && !this.mIsAnimatingOut && child.getVisibility() == View.VISIBLE) {
		// User scrolled down and the FAB is currently visible -> hide the FAB
		animateOut(child);
	} else if (dyConsumed < 0 && child.getVisibility() != View.VISIBLE) {
		// User scrolled up and the FAB is currently not visible -> show the FAB
		animateIn(child);
	}
}


// Same animation that FloatingActionButton.Behavior uses to hide the FAB when the AppBarLayout exits
private void animateOut(final FloatingActionButton button) {
	if (Build.VERSION.SDK_INT >= 14) {
		ViewCompat.animate(button)
			.translationY(button.getHeight() + getMarginBottom(button))
			.setInterpolator(INTERPOLATOR)
			.withLayer()
			.setListener(new ViewPropertyAnimatorListener() {
				public void onAnimationStart(View view) {
					ScrollAwareFABBehavior.this.mIsAnimatingOut = true;
				}


				public void onAnimationCancel(View view) {
					ScrollAwareFABBehavior.this.mIsAnimatingOut = false;
				}


				public void onAnimationEnd(View view) {
					ScrollAwareFABBehavior.this.mIsAnimatingOut = false;
					view.setVisibility(View.GONE);
				}
			})
			.start();
	} else {
		Animation anim = AnimationUtils.loadAnimation(button.getContext(),
			android.support.design.R.anim.design_fab_out);
		anim.setInterpolator(INTERPOLATOR);
		anim.setDuration(200L);
		anim.setAnimationListener(new Animation.AnimationListener() {
			public void onAnimationStart(Animation animation) {
				ScrollAwareFABBehavior.this.mIsAnimatingOut = true;
			}


			public void onAnimationEnd(Animation animation) {
				ScrollAwareFABBehavior.this.mIsAnimatingOut = false;
				button.setVisibility(View.GONE);
			}


			@Override public void onAnimationRepeat(final Animation animation) {
			}
		});
		button.startAnimation(anim);
	}
}


// Same animation that FloatingActionButton.Behavior uses to show the FAB when the AppBarLayout enters
private void animateIn(FloatingActionButton button) {
	button.setVisibility(View.VISIBLE);
	if (Build.VERSION.SDK_INT >= 14) {
		ViewCompat.animate(button)
			.translationY(0)
			.setInterpolator(INTERPOLATOR)
			.withLayer()
			.setListener(null)
			.start();
	} else {
		Animation anim = AnimationUtils.loadAnimation(button.getContext(),
			android.support.design.R.anim.design_fab_in);
		anim.setDuration(200L);
		anim.setInterpolator(INTERPOLATOR);
		button.startAnimation(anim);
	}
}

private int getMarginBottom(View v) {
	int marginBottom = 0;
	final ViewGroup.LayoutParams layoutParams = v.getLayoutParams();
	if (layoutParams instanceof ViewGroup.MarginLayoutParams) {
		marginBottom = ((ViewGroup.MarginLayoutParams) layoutParams).bottomMargin;
	}
	return marginBottom;
}
```

效果如图所示：


![竖直平移](http://upload-images.jianshu.io/upload_images/1159224-b819b23de13e19c4.gif?imageMogr2/auto-orient/strip)


![缩放](http://upload-images.jianshu.io/upload_images/1159224-a435076eda79bd30.gif?imageMogr2/auto-orient/strip)

另外，`FAB`的xml中有几个属性要说下，`app:backgroundTint`是设置正常状态下的背景颜色，`app:rippleColor`是设置点击状态下的波纹颜色，`app:srcCompat`是设置它里面的图片。


## Snackbar

`Snackbar`的用法也很简单：

```java
Snackbar.make(view, "Replace with your own action", Snackbar.LENGTH_LONG).show();
或者
Snackbar.make(view, "Replace with your own action", Snackbar.LENGTH_LONG)
		.setAction("Action", new View.OnClickListener() {
			@Override public void onClick(View view) {
				// do sth
			}
		})
		.show();
```

谷歌官方推荐`Snackbar`使用时外层包裹一个`CoordinatorLayout`作为`parent`，以确保它和其他组件的交互正常，它本质上是一个显示在屏幕最上层的`FrameLayout`，看源码可知，`Snackbar`继承自`BaseTransientBottomBar`，而`BaseTransientBottomBar`的构造方法中可以看到

```java
mView = (SnackbarBaseLayout) inflater.inflate(R.layout.design_layout_snackbar, mTargetParent, false);
mView.addView(content);
```

`Snackbar`只是一个容器`view`，`content`才是它的内容，再看看`design_layout_snackbar.xml`，目录在`sdk\extras\android\m2repository\com\android\support\design\xx\design-xx.aar`，解压即可。

```xml
<?xml version="1.0" encoding="utf-8"?>
<view xmlns:android="http://schemas.android.com/apk/res/android"
      class="android.support.design.widget.Snackbar$SnackbarLayout"
      android:layout_width="match_parent"
      android:layout_height="wrap_content"
      android:layout_gravity="bottom"
      android:theme="@style/ThemeOverlay.AppCompat.Dark"
      style="@style/Widget.Design.Snackbar" />
```

我们看到class这个属性，它表示这个`view`实际上是`Snackbar`中的内部类`SnackbarLayout`：

```java
/**
 * @hide
 *
 * Note: this class is here to provide backwards-compatible way for apps written before
 * the existence of the base {@link BaseTransientBottomBar} class.
 */
@RestrictTo(LIBRARY_GROUP)
public static final class SnackbarLayout extends BaseTransientBottomBar.SnackbarBaseLayout {
	public SnackbarLayout(Context context) {
		super(context);
	}

	public SnackbarLayout(Context context, AttributeSet attrs) {
		super(context, attrs);
	}
}
```

`SnackbarBaseLayout `正是一个`FrameLayout`，上面说`content`才是它内容布局，是在`make`方法中生成的：

```java
@NonNull
public static Snackbar make(@NonNull View view, @NonNull CharSequence text,
		@Duration int duration) {
	final ViewGroup parent = findSuitableParent(view);
	final LayoutInflater inflater = LayoutInflater.from(parent.getContext());
	final SnackbarContentLayout content = (SnackbarContentLayout) inflater.inflate(
					R.layout.design_layout_snackbar_include, parent, false);
	final Snackbar snackbar = new Snackbar(parent, content, content);
	snackbar.setText(text);
	snackbar.setDuration(duration);
	return snackbar;
}
```

内容布局就是一个`SnackbarContentLayout `类，而它是`LinearLayout`的子类，也就是一个线性布局。再看看`design_layout_snackbar_include.xml`它的内部布局：

```xml
<?xml version="1.0" encoding="utf-8"?>
<merge xmlns:android="http://schemas.android.com/apk/res/android">

    <TextView
            android:id="@+id/snackbar_text"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:paddingTop="@dimen/design_snackbar_padding_vertical"
            android:paddingBottom="@dimen/design_snackbar_padding_vertical"
            android:paddingLeft="@dimen/design_snackbar_padding_horizontal"
            android:paddingRight="@dimen/design_snackbar_padding_horizontal"
            android:textAppearance="@style/TextAppearance.Design.Snackbar.Message"
            android:maxLines="@integer/design_snackbar_text_max_lines"
            android:layout_gravity="center_vertical|left|start"
            android:ellipsize="end"
            android:textAlignment="viewStart"/>

    <Button
            android:id="@+id/snackbar_action"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginLeft="@dimen/design_snackbar_extra_spacing_horizontal"
            android:layout_marginStart="@dimen/design_snackbar_extra_spacing_horizontal"
            android:layout_gravity="center_vertical|right|end"
            android:minWidth="48dp"
            android:visibility="gone"
            android:textColor="?attr/colorAccent"
            style="?attr/borderlessButtonStyle"/>

</merge>
```

所以左边是一个显示`message`的`TextView`，右边是提供点击事件的`Button`，知道了它组件的id，那我们给它设置自己喜欢的背景色和文字颜色都可以了：

```java
Snackbar snackbar = Snackbar.make(view, "Replace with your own action", Snackbar.LENGTH_LONG);
snackbar.setAction("点我点我", new View.OnClickListener() {
	@Override public void onClick(View view) {
		// do your sth
	}
});
View snackbarView = snackbar.getView();
// 修改snackbar的背景颜色
snackbarView.setBackgroundColor(getResources().getColor(R.color.colorPrimary));
// 修改snackbar中message文字的颜色
((TextView) snackbarView.findViewById(android.support.design.R.id.snackbar_text)).setTextColor(getResources().getColor(R.color.colorAccent));
// 修改snackbar中action按钮文字的颜色
((Button) snackbarView.findViewById(android.support.design.R.id.snackbar_action)).setTextColor(getResources().getColor(R.color.colorAccent));
snackbar.show();
```

效果如图：

![snackbar.gif](http://upload-images.jianshu.io/upload_images/1159224-3e318a1f725888fd.gif?imageMogr2/auto-orient/strip)

`Snackbar`每次只能显示一个，原因我们可以看下源码中`Snackbar`的`show()`方法：

```java
/**
 * Show the {@link BaseTransientBottomBar}.
 */
public void show() {
	SnackbarManager.getInstance().show(mDuration, mManagerCallback);
}
```

它的显示和隐藏是有`SnackbarManager`来控制的，看到`SnackbarManager`这种写法，我已经猜出来它是个单例了：

```java
private static SnackbarManager sSnackbarManager;

static SnackbarManager getInstance() {
	if (sSnackbarManager == null) {
		sSnackbarManager = new SnackbarManager();
	}
	return sSnackbarManager;
}
```

再来看`SnackbarManager`中的`show()`方法实现：

```java
public void show(int duration, Callback callback) {
	synchronized (mLock) {
		if (isCurrentSnackbarLocked(callback)) {
			// Means that the callback is already in the queue. We'll just update the duration
			mCurrentSnackbar.duration = duration;

			// If this is the Snackbar currently being shown, call re-schedule it's
			// timeout
			mHandler.removeCallbacksAndMessages(mCurrentSnackbar);
			scheduleTimeoutLocked(mCurrentSnackbar);
			return;
		} else if (isNextSnackbarLocked(callback)) {
			// We'll just update the duration
			mNextSnackbar.duration = duration;
		} else {
			// Else, we need to create a new record and queue it
			mNextSnackbar = new SnackbarRecord(duration, callback);
		}

		if (mCurrentSnackbar != null && cancelSnackbarLocked(mCurrentSnackbar,
				Snackbar.Callback.DISMISS_EVENT_CONSECUTIVE)) {
			// If we currently have a Snackbar, try and cancel it and wait in line
			return;
		} else {
			// Clear out the current snackbar
			mCurrentSnackbar = null;
			// Otherwise, just show it now
			showNextSnackbarLocked();
		}
	}
}
```

可以看到这个方法是同步加锁的，只有当前没有`Snackbar`显示时才会让其显示，否则会先`cancelSnackbarLocked`当前的`Snackbar`的回调和信息，然后调用`onDismissed`方法，等这条消失后再`showNextSnackbarLocked`下一个。它用两个`SnackbarRecord `类型的变量`mCurrentSnackbar`和`mNextSnackbar`来维持了显示和待显示的`Snackbar`队列。

它的作用其实和`Toast`类似，也是给用户一些友好的提示信息，不过它比`Toast`更加丰富，不仅可以显示`message`还可以`setAction`设置Snackbar右侧按钮，增加进行交互事件。

## TextInputLayout

`AndroidStudio`默认新建的`LoginActivity`中就有使用`TextInputLayout`，我们来看看它的xml布局：

```xml
<LinearLayout
	android:id="@+id/email_login_form"
	android:layout_width="match_parent"
	android:layout_height="wrap_content"
	android:orientation="vertical">

	<android.support.design.widget.TextInputLayout
		android:layout_width="match_parent"
		android:layout_height="wrap_content">

		<AutoCompleteTextView
			android:id="@+id/email"
			android:layout_width="match_parent"
			android:layout_height="wrap_content"
			android:hint="@string/prompt_email"
			android:inputType="textEmailAddress"
			android:maxLines="1"/>

	</android.support.design.widget.TextInputLayout>

	<android.support.design.widget.TextInputLayout
		android:layout_width="match_parent"
		android:layout_height="wrap_content">

		<EditText
			android:id="@+id/password"
			android:layout_width="match_parent"
			android:layout_height="wrap_content"
			android:hint="@string/prompt_password"
			android:imeActionId="@+id/login"
			android:imeActionLabel="@string/action_sign_in_short"
			android:imeOptions="actionUnspecified"
			android:inputType="textPassword"
			android:maxLines="1"
			android:singleLine="true"/>

	</android.support.design.widget.TextInputLayout>

	<Button
		style="?android:textAppearanceSmall"
		android:id="@+id/email_sign_in_button"
		android:layout_width="match_parent"
		android:layout_height="wrap_content"
		android:layout_marginTop="16dp"
		android:text="@string/action_sign_in"
		android:textStyle="bold"/>

</LinearLayout>
```

从布局上就可以看出`TextInputLayout`实际上就是一个`ViewGroup`，它是继承自`LinearLayout`，且排列方式是竖直方向：

```java
setOrientation(VERTICAL);
```

它的构造方法中，先`addView`一个`FrameLayout`：

```java
mInputFrame = new FrameLayout(context);
mInputFrame.setAddStatesFromChildren(true);
addView(mInputFrame);
```

然后重写了`addView`方法，如果是`EditText`控件，就添加到`mInputFrame`中：

```java
@Override
public void addView(View child, int index, final ViewGroup.LayoutParams params) {
	if (child instanceof EditText) {
		mInputFrame.addView(child, new FrameLayout.LayoutParams(params));

		// Now use the EditText's LayoutParams as our own and update them to make enough space
		// for the label
		mInputFrame.setLayoutParams(params);
		updateInputLayoutMargins();

		setEditText((EditText) child);
	} else {
		// Carry on adding the View...
		super.addView(child, index, params);
	}
}
```

其中`setEditText`方法中给`EditText`设置了监听：

```java
// Add a TextWatcher so that we know when the text input has changed
mEditText.addTextChangedListener(new TextWatcher() {
	@Override
	public void afterTextChanged(Editable s) {
		updateLabelState(true);
		if (mCounterEnabled) {
			updateCounter(s.length());
		}
	}

	@Override
	public void beforeTextChanged(CharSequence s, int start, int count, int after) {}

	@Override
	public void onTextChanged(CharSequence s, int start, int before, int count) {}
});
```

并且在方法最后调用了下面这个方法：

```java
// Update the label visibility with no animation
updateLabelState(false);
```

在`EditText`的监听方法中也有调用该方法，它是根据`EditText`是否获取了焦点，是否有文字等判断提示文字的展开或折叠：

```java
void updateLabelState(boolean animate) {
	final boolean isEnabled = isEnabled();
	final boolean hasText = mEditText != null && !TextUtils.isEmpty(mEditText.getText());
	final boolean isFocused = arrayContains(getDrawableState(), android.R.attr.state_focused);
	final boolean isErrorShowing = !TextUtils.isEmpty(getError());

	if (mDefaultTextColor != null) {
		mCollapsingTextHelper.setExpandedTextColor(mDefaultTextColor);
	}

	if (isEnabled && mCounterOverflowed && mCounterView != null) {
		mCollapsingTextHelper.setCollapsedTextColor(mCounterView.getTextColors());
	} else if (isEnabled && isFocused && mFocusedTextColor != null) {
		mCollapsingTextHelper.setCollapsedTextColor(mFocusedTextColor);
	} else if (mDefaultTextColor != null) {
		mCollapsingTextHelper.setCollapsedTextColor(mDefaultTextColor);
	}

	if (hasText || (isEnabled() && (isFocused || isErrorShowing))) {
		// We should be showing the label so do so if it isn't already
		collapseHint(animate);
	} else {
		// We should not be showing the label so hide it
		expandHint(animate);
	}
}
```

无论是展开还是折叠hint文字，最终都会调用`mCollapsingTextHelper`的`setExpansionFraction`方法：

```java
private void collapseHint(boolean animate) {
	if (mAnimator != null && mAnimator.isRunning()) {
		mAnimator.cancel();
	}
	if (animate && mHintAnimationEnabled) {
		animateToExpansionFraction(1f);
	} else {
		mCollapsingTextHelper.setExpansionFraction(1f);
	}
	mHintExpanded = false;
}

private void expandHint(boolean animate) {
	if (mAnimator != null && mAnimator.isRunning()) {
		mAnimator.cancel();
	}
	if (animate && mHintAnimationEnabled) {
		animateToExpansionFraction(0f);
	} else {
		mCollapsingTextHelper.setExpansionFraction(0f);
	}
	mHintExpanded = true;
}

private void animateToExpansionFraction(final float target) {
	if (mCollapsingTextHelper.getExpansionFraction() == target) {
		return;
	}
	if (mAnimator == null) {
		mAnimator = ViewUtils.createAnimator();
		mAnimator.setInterpolator(AnimationUtils.LINEAR_INTERPOLATOR);
		mAnimator.setDuration(ANIMATION_DURATION);
		mAnimator.addUpdateListener(new ValueAnimatorCompat.AnimatorUpdateListener() {
			@Override
			public void onAnimationUpdate(ValueAnimatorCompat animator) {
				mCollapsingTextHelper.setExpansionFraction(animator.getAnimatedFloatValue());
			}
		});
	}
	mAnimator.setFloatValues(mCollapsingTextHelper.getExpansionFraction(), target);
	mAnimator.start();
}
```
`mCollapsingTextHelper`是提示文字的折叠辅助类，它计算`EditText`的文字展开和折叠时边界的尺寸，`setExpansionFraction`方法是根据传递的分数来计算文字当前展开或折叠的程度。

```java
/**
 * Set the value indicating the current scroll value. This decides how much of the
 * background will be displayed, as well as the title metrics/positioning.
 *
 * A value of {@code 0.0} indicates that the layout is fully expanded.
 * A value of {@code 1.0} indicates that the layout is fully collapsed.
 */
void setExpansionFraction(float fraction) {
	fraction = MathUtils.constrain(fraction, 0f, 1f);

	if (fraction != mExpandedFraction) {
		mExpandedFraction = fraction;
		calculateCurrentOffsets();
	}
}
```

完成计算后调用`postInvalidateOnAnimation`方法进行重绘。

## CollapsingToolbarLayout

用`AndroidStudio`工具新建的`ScrollingActivity`模板其实就是一个`CollapsingToolbarLayout`的使用场景，它默认的xml布局：

```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.design.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fitsSystemWindows="true"
    tools:context="com.xx.design.CollapsingToolbarLayoutActivity">

    <android.support.design.widget.AppBarLayout
        android:id="@+id/app_bar"
        android:layout_width="match_parent"
        android:layout_height="@dimen/app_bar_height"
        android:fitsSystemWindows="true"
        android:theme="@style/AppTheme.AppBarOverlay">

        <android.support.design.widget.CollapsingToolbarLayout
            android:id="@+id/toolbar_layout"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:fitsSystemWindows="true"
            app:layout_scrollFlags="scroll|exitUntilCollapsed"
            app:contentScrim="?attr/colorPrimary">

            <android.support.v7.widget.Toolbar
                android:id="@+id/toolbar"
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                app:layout_collapseMode="pin"
                app:popupTheme="@style/AppTheme.PopupOverlay"/>

        </android.support.design.widget.CollapsingToolbarLayout>
    </android.support.design.widget.AppBarLayout>

    <include layout="@layout/content_collapsing_toolbar_layout"/>

    <android.support.design.widget.FloatingActionButton
        android:id="@+id/fab"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_margin="@dimen/fab_margin"
        app:layout_anchor="@id/app_bar"
        app:layout_anchorGravity="bottom|end"
        app:srcCompat="@android:drawable/ic_dialog_email"/>

</android.support.design.widget.CoordinatorLayout>
```

在`Toolbar`外面还包裹了一层`CollapsingToolbarLayout`，它实际是`FrameLayout`。`CollapsingToolbarLayout`可以通过`app:contentScrim`设置折叠时工具栏布局的颜色，通过`app:statusBarScrim`设置折叠时状态栏的颜色。默认`contentScrim`是`colorPrimary`的色值，`statusBarScrim`是`colorPrimaryDark`的色值。`CollapsingToolbarLayout`的子布局有3种折叠模式`app:layout_collapseMode`，none这个是默认属性，布局将正常显示，没有折叠的行为，pin表示`CollapsingToolbarLayout`折叠后，此布局将固定在顶部，parallax表示`CollapsingToolbarLayout`折叠时，此布局也会有视差折叠效果。

再看`FAB`的位置，是因为它设置了这两个属性：`app:layout_anchor="@id/app_bar"`和`app:layout_anchorGravity="bottom|end"`。

一般的做法是当`Toolbar`展开时显示一张背景图片，只需要在`Toolbar`后面加一个`ImageView`即可：

```xml
<android.support.design.widget.CollapsingToolbarLayout
	android:id="@+id/toolbar_layout"
	android:layout_width="match_parent"
	android:layout_height="match_parent"
	android:fitsSystemWindows="true"
	app:layout_scrollFlags="scroll|exitUntilCollapsed"
	app:contentScrim="?attr/colorPrimary">

	<!--封面图片-->
	<ImageView
		android:id="@+id/imageview"
		android:layout_width="match_parent"
		android:layout_height="match_parent"
		android:scaleType="centerCrop"
		android:src="@mipmap/toolbar"
		app:layout_collapseMode="pin"
		android:fitsSystemWindows="true"/>

	<android.support.v7.widget.Toolbar
		android:id="@+id/toolbar"
		android:layout_width="match_parent"
		android:layout_height="?attr/actionBarSize"
		app:layout_collapseMode="pin"
		app:popupTheme="@style/AppTheme.PopupOverlay">

	</android.support.v7.widget.Toolbar>

</android.support.design.widget.CollapsingToolbarLayout>
```


![CollapsingToolbarLayout](http://upload-images.jianshu.io/upload_images/1159224-f4c87cd5672d2f9d.gif?imageMogr2/auto-orient/strip)

看`CollapsingToolbarLayout`的源码发现，它里面也用到了`CollapsingTextHelper`类，就是在`Toolbar`上的文字title的展开和折叠，其实是和`TextInputLayout`一样的。当它的外层是`AppBarLayout`包裹时，可以监听其竖直方向上偏移量的变化：

```java
private class OffsetUpdateListener implements AppBarLayout.OnOffsetChangedListener {
	OffsetUpdateListener() {
	}

	@Override
	public void onOffsetChanged(AppBarLayout layout, int verticalOffset) {
		mCurrentOffset = verticalOffset;

		final int insetTop = mLastInsets != null ? mLastInsets.getSystemWindowInsetTop() : 0;

		for (int i = 0, z = getChildCount(); i < z; i++) {
			final View child = getChildAt(i);
			final LayoutParams lp = (LayoutParams) child.getLayoutParams();
			final ViewOffsetHelper offsetHelper = getViewOffsetHelper(child);

			switch (lp.mCollapseMode) {
				case LayoutParams.COLLAPSE_MODE_PIN:
					offsetHelper.setTopAndBottomOffset(
							constrain(-verticalOffset, 0, getMaxOffsetForPinChild(child)));
					break;
				case LayoutParams.COLLAPSE_MODE_PARALLAX:
					offsetHelper.setTopAndBottomOffset(
							Math.round(-verticalOffset * lp.mParallaxMult));
					break;
			}
		}

		// Show or hide the scrims if needed
		updateScrimVisibility();

		if (mStatusBarScrim != null && insetTop > 0) {
			ViewCompat.postInvalidateOnAnimation(CollapsingToolbarLayout.this);
		}

		// Update the collapsing text's fraction
		final int expandRange = getHeight() - ViewCompat.getMinimumHeight(
				CollapsingToolbarLayout.this) - insetTop;
		mCollapsingTextHelper.setExpansionFraction(
				Math.abs(verticalOffset) / (float) expandRange);
	}
}
```

增加监听：
```java
((AppBarLayout) parent).addOnOffsetChangedListener(mOnOffsetChangedListener);
```

> 另外需要注意的是下面滑动的部分，是`NestedScrollView`而不能时`ScrollView`，因为前者才实现了`NestedScrollingParent`接口，和`CoordinatorLayout`结合使用滚动时才会有动画的效果。

