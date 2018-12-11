---
layout: post
title: Android动画之帧动画和补间动画
category: blog
tags: 帧动画、补间动画
---



## Android动画分类

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Android动画可以说是Android里面比较重要的部分，它分为三种：`帧动画`、`补间动画`和`属性动画`。由于帧动画和补间动画相对属性动画来说，简单一些，所以把这俩放在一起记录，属性动画单独拿出来记录。

## 帧动画

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;帧动画其实就是通过顺序播放一系列图片从而产生动画的效果，就像电视画面一样，可以简单的理解为图片切换，Android系统提供了一个类`AnimationDrawable`来使用帧动画，使用起来也比较简单。

首先需要通过xml来定义一个AnimationDrawable：

```java
<?xml version="1.0" encoding="utf-8"?>
<animation-list xmlns:android="http://schemas.android.com/apk/res/android"
    android:oneshot="false">
    
    <item android:drawable="@drawable/loading1" android:duration="500" />
    <item android:drawable="@drawable/loading2" android:duration="500" />
    <item android:drawable="@drawable/loading3" android:duration="500" />
    <item android:drawable="@drawable/loading4" android:duration="500" />
    <item android:drawable="@drawable/loading5" android:duration="500" />
    <item android:drawable="@drawable/loading6" android:duration="500" />
    <item android:drawable="@drawable/loading7" android:duration="500" />
    <item android:drawable="@drawable/loading8" android:duration="500" />

</animation-list>
```

其中，android:onshot如果为true，表示动画只播放一次停止在最后一帧上，如果设置为false表示动画循环播放。

然后将上述的Drawable作为View的背景并通过Drawable来播放动画即可。

```java
TextView textview = (TextView) findViewById(R.id.textview);
textview.setBackgroundResource(R.drawable.animation_frame);
AnimationDrawable drawable = (AnimationDrawable) textview.getBackground();
drawable.start();
```

> 帧动画的使用比较简单，但是如果图片很多而且很大时，容易引起OOM，所以在使用帧动画时应该尽量避免使用过多尺寸较大的图片。


## 补间动画

补间动画也叫View动画，它的作用对象是View，它只支持平移动画、缩放动画、旋转动画和渐变动画这四种动画效果。这四种变换效果分别对应着Animation的四个子类：TranslateAnimation、ScaleAnimation、RotationAnimation和AlphaAnimation。和属性动画一样，这四种动画既可以通过代码方式动态创建，也可以通过xml方式来定义，他们在xml中分别对应的标签是`<translate>`、`<scale>`、`<rotate>`、`<alpha>`。其中渐变动画指的是改变View的透明度。

### 通过XML定义

首先需要创建动画的XML，这个xml文件存放在res/anim/目录下：

```java
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="2000"
    android:fillAfter="true"
    android:fillBefore="false"
    android:interpolator="@android:anim/accelerate_decelerate_interpolator"
    android:repeatMode="reverse"
    android:shareInterpolator="true"
    android:startOffset="0">

    <!-- 平移动画 -->
    <translate
        android:fromXDelta="0"
        android:fromYDelta="0"
        android:toXDelta="10"
        android:toYDelta="10" />

    <!-- 缩放动画 -->
    <scale
        android:fromXScale="1"
        android:fromYScale="1"
        android:toXScale="2.5"
        android:toYScale="2.5" />

    <!-- 旋转动画 -->
    <rotate
        android:fromDegrees="0"
        android:toDegrees="360" />

    <!-- 渐变动画 -->
    <alpha
        android:fromAlpha="1"
        android:toAlpha="0.5" />

</set>
```

然后在代码中应用上面的动画：

```java
final Animation animation = AnimationUtils.loadAnimation(this, R.anim.animation_set);

findViewById(R.id.textview).setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        v.startAnimation(animation);
    }
});
```

效果如图：

![补间动画](https://static.oschina.net/uploads/img/201704/16144153_sDyE.gif "补间动画")

上面的xml中是一系列动画组合在一起的，但每种动画标签都可以单独使用。`<set>`标签表示动画集合，对应`AnimationSet`类，它可以包含若干个子动画，并且在它内部还可以嵌套其他的动画集合。

> 千万不要把这里的AnimationSet和属性动画中的动画集合AnimatorSet搞混，虽然都是动画集合，但AnimationSet是android.view.animation包下的类，而AnimatorSet是android.animation包下的类。前者只对View起作用。

再来看看xml中各个标签以及属性的含义：

- android:interpolator表示动画集合采用的插值器，不指定的话默认就是accelerate_decelerate_interpolator加减速插值器。

- android:shareInterpolator表示集合中的动画是否和集合共享同一个插值器，如果集合不指定插值器，那么子动画就需要单独指定所需要的插值器或者使用默认值。

- android:duration表示动画执行的时间

- android:fillAfter表示动画结束以后View是否停留在结束位置，true表示停留在结束位置，false则表示回到View原始的位置

- android:startOffset表示动画延迟执行的时间

- android:zAdjustment允许在动画播放期间，调整播放内容在Z轴方向的顺序：

    - normal（0）：正在播放的动画内容保持当前的Z轴顺序

    - top（1）：在动画播放期间，强制把当前播放的内容放到其他内容的上面

    - bottom（-1）：在动画播放期间，强制把当前播放的内容放到其他内容之下

`<translate>`标签表示平移动画，它的作用效果是使View在水平和竖直方向上移动，它的下列属性含义：

- android:fromXDelta    水平方向的起始值
- android:fromYDelta    竖直方向的起始值
- android:toXDelta        水平方向的结束值
- android:toYDelta        竖直方向的结束值

'<scale>'标签表示缩放动画，它的所用效果是使View能够放大缩小，它的下列属性含义：

- android:fromXScale    水平方向缩放的起始值 
- android:fromYScale    竖直方向缩放的起始值 
- android:toXScale    水平方向缩放的结束值
- android:toYScale    竖直方向缩放的结束值

另外，如果你想指定缩放的轴点，需要用这两个属性：

- android:pivotX    缩放轴点的x坐标
- android:pivotY    缩放轴点的y坐标

> 不指定轴点时，缩放动画会以View的左上角为轴点进行缩放。

`<rotate>`标签表示旋转动画，它的作用效果是使View进行旋转。它的下列属性的含义：

- android:fromDegrees    旋转开始的角度
- android:toDegrees    旋转结束的角度

和scale动画一样，旋转动画也可以用android:pivotX和android:pivotY来指定旋转的轴点，默认也是View的左上角。

`<alpha>`标签表示渐变动画，它的作用效果是改变View的透明度，其下列属性含义：

- android:fromAlpha    起始透明度值
- android:toAlpha    结束透明度值

> 透明度的值从0到1之间。


### 在代码中创建补间动画

```java
final TranslateAnimation translateAnimation = new TranslateAnimation(0, 200, 0, 200);
translateAnimation.setDuration(2000);
translateAnimation.setInterpolator(new AccelerateInterpolator());
translateAnimation.setFillAfter(true);

findViewById(R.id.textview).setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        v.startAnimation(translateAnimation);
    }
});
```

但是这种方式好像有个缺陷，比如我有多个动画想要同时执行，而View的startAnimation方法只能开始一个动画。比如下面这样写：

```java
final TranslateAnimation translateAnimation = new TranslateAnimation(0, 200, 0, 200);
translateAnimation.setDuration(2000);
translateAnimation.setInterpolator(new AccelerateInterpolator());
translateAnimation.setFillAfter(true);

final ScaleAnimation scaleAnimation = new ScaleAnimation(1, 1.5f, 1, 2);
scaleAnimation.setDuration(2000);

findViewById(R.id.textview).setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        v.startAnimation(translateAnimation);
        v.startAnimation(scaleAnimation);
    }
});
```

View就只会执行后面的缩放动画，如果要多个动画同时执行，就只能用xml的方式定义，然后AnimationUtils.loadAnimation加载出来了。这里如果有错，希望大家指正。

### 监听动画的执行

```java
TranslateAnimation translateAnimation = new TranslateAnimation(0, 200, 0, 200);
        
Animation.AnimationListener animationListener = new Animation.AnimationListener() {
    @Override
    public void onAnimationStart(Animation animation) {}

    @Override
    public void onAnimationEnd(Animation animation) {}

    @Override
    public void onAnimationRepeat(Animation animation) {}
};
translateAnimation.setAnimationListener(animationListener);
```


## 针对ViewGroup的动画

帧动画和补间动画都是针对View的动画，而`LayoutAnimation`和`LayoutAnimationController`则是针对ViewGroup的做动画的两个类。LayoutAnimation为ViewGroup指定一个动画，这样当它的子View出场时都会具有这种动画效果，比较常见的有ListView和GridView还有RecyclerView的item的出场效果。

在xml中定义LayoutAnimation：

```java
<?xml version="1.0" encoding="utf-8"?>
<layoutAnimation xmlns:android="http://schemas.android.com/apk/res/android"
    android:animation="@anim/animation_item"
    android:animationOrder="normal"
    android:delay="0.5">

</layoutAnimation>
```

android:animation表示作用在子View上的动画效果，它里面的文件animation_item也是定义在anim目录下的：

```java
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="500"
    android:interpolator="@android:anim/accelerate_interpolator"
    android:shareInterpolator="true">

    <translate
        android:fromXDelta="-500"
        android:toXDelta="0" />

    <alpha
        android:fromAlpha="0"
        android:toAlpha="1" />

</set>
```

android:animationOrder表示子View播放动画的顺序：

normal[顺序出场] | random[随机出场] | reverse[逆向出场]

android:delay表示子View开始动画的延迟时间，注意这里的时间单位既不是秒也不是毫秒，比如我们这里子View动画的duration是500毫秒，那么这个dalay0.5表示每个子元素都需要延迟250毫秒才开始做动画，第一个item延迟250毫秒，第二延迟500，第三个750...

android:interpolator表示所有子View执行动画的插值器

在xml中为ViewGroup指定andorid:layoutAnimation属性：以RecyclerView为例

```java
<android.support.v7.widget.RecyclerView
    android:id="@+id/recyclerview"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:layoutAnimation="@anim/layout_animation">

</android.support.v7.widget.RecyclerView>
```

```java
RecyclerView mRecyclerView = (RecyclerView) findViewById(R.id.recyclerview);

mRecyclerView.setLayoutManager(new LinearLayoutManager(this));
List<String> titles = new ArrayList<>();
for (int i = 0; i < 50; i++) {
    titles.add("LayoutAnimation的运用" + i);
}
mRecyclerView.setAdapter(new RecyclerViewAdapter(this, titles));
```

适配器代码：

```java
package com.shenhuniurou.tweenanimation;

import android.content.Context;
import android.support.v7.widget.RecyclerView;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;

import java.util.List;

/**
 * Created by shenhuniurou on 2017/4/16.
 */

public class RecyclerViewAdapter extends RecyclerView.Adapter<RecyclerViewAdapter.TextViewHolder> {

    private final LayoutInflater mLayoutInflater;
    private final Context mContext;
    private List<String> mTitles;
    private OnItemClickListener listener;

    interface OnItemClickListener {
        public void OnItemClick(int position);
    }

    public void setOnItemClickListener(OnItemClickListener listener) {
        this.listener = listener;
    }

    public RecyclerViewAdapter(Context context, List<String> titles) {
        mTitles = titles;
        mContext = context;
        mLayoutInflater = LayoutInflater.from(context);
    }

    @Override
    public TextViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        return new TextViewHolder(mLayoutInflater.inflate(R.layout.item_text, parent, false));
    }

    @Override
    public void onBindViewHolder(TextViewHolder holder, int position) {
        holder.mTextView.setText(mTitles.get(position));
    }

    @Override
    public int getItemCount() {
        return mTitles == null ? 0 : mTitles.size();
    }

    public class TextViewHolder extends RecyclerView.ViewHolder {

        TextView mTextView;

        TextViewHolder(View view) {
            super(view);
            mTextView = (TextView) view.findViewById(R.id.textview);
        }
    }

}
```

最后运行的效果如图：

![LayoutAnimation](https://static.oschina.net/uploads/img/201704/16181619_FB4q.gif "LayoutAnimation")

除了在XML中指定LayoutAnimation外，还可以在代码中通过LayoutAnimationController类来实现：

```java
RecyclerView mRecyclerView = (RecyclerView) findViewById(R.id.recyclerview);

Animation animation = AnimationUtils.loadAnimation(this, R.anim.animation_item);
LayoutAnimationController controller = new LayoutAnimationController(animation);
controller.setDelay(0.5);
controller.setOrder(LayoutAnimationController.ORDER_NORMAL);
mRecyclerView.setLayoutAnimation(controller);

mRecyclerView.setLayoutManager(new LinearLayoutManager(this));
List<String> titles = new ArrayList<>();
for (int i = 0; i < 50; i++) {
    titles.add("LayoutAnimation的运用" + i);
}
mRecyclerView.setAdapter(new RecyclerViewAdapter(this, titles));
```

`LayoutAnimationController`适用于ListView，对于GridView就必须适用`GridLayoutAnimationController`，如果是RecyclerView，当它的LayoutManager是LinearLayourManager时就可以使用LayoutAnimationController，但如果设置成GridLayoutManager，就不能使用LayoutAnimationController了，否则会出现类转换异常。


## 转场动画

补间动画还可以作为转场动画使用，即Activity的切换动画效果。Android系统本身有默认的切换动画，但是我们可以自定义。

主要用到的方法是overridePendingTransition(int enterAnim, int exitAnim);这个方法必须在调用startActivity或者finish后被调用才能生效。enterAnim是Activity被打开时调用的动画资源，exitAnim是Activity退出时的动画资源。系统内自带的一些可以用`android.R.anim.fade_in`这种形式调用。下面是我自定义的一个转场动画资源：

enter_anim.xml

```java
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="500">
    <translate
        android:duration="@android:integer/config_mediumAnimTime"
        android:fromXDelta="100%p"
        android:toXDelta="0" />
    <alpha
        android:duration="@android:integer/config_mediumAnimTime"
        android:fromAlpha="0.0"
        android:toAlpha="1.0" />

    <scale
        android:fromXScale="0.8"
        android:fromYScale="0.8"
        android:pivotY="50%"
        android:toXScale="1"
        android:toYScale="1" />
</set>
```

exit_anim.xml

```java
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="500">
    <translate
        android:duration="@android:integer/config_mediumAnimTime"
        android:fromXDelta="0"
        android:toXDelta="-100%p" />
    <alpha
        android:duration="@android:integer/config_mediumAnimTime"
        android:fromAlpha="1.0"
        android:toAlpha="0.0" />

    <scale
        android:fromXScale="1"
        android:fromYScale="1"
        android:pivotY="50%"
        android:toXScale="0.5"
        android:toYScale="0.5" />
</set>
```

效果如图：

![转场动画](https://static.oschina.net/uploads/img/201704/16190257_E1Ny.gif "转场动画")

只要掌握了补间动画的这些知识，完全可以根据自己的需求的审美自定义出符合你要求的转场动画。

转场动画除了可以调用overridePendingTransition这个方法来实现，还可以通过配置style来实现：

```java
<style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
    <!-- Customize your theme here. -->
    <item name="colorPrimary">@color/colorPrimary</item>
    <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
    <item name="colorAccent">@color/colorAccent</item>

    <item name="android:windowAnimationStyle">@style/activityAnimation</item>

</style>

<style name="activityAnimation" parent="@android:style/Animation.Activity">
    <item name="android:activityOpenEnterAnimation"></item>
    <item name="android:activityOpenExitAnimation"></item>
    <item name="android:activityCloseEnterAnimation"></item>
    <item name="android:activityCloseExitAnimation"></item>
</style>
```

就可以实现和上图一样的切换效果了。

> 开发过程中遇到一个奇怪的问题，当View进行缩放动画后，有效点击的部分还是View原来的位置，而在View上其他的部位会出现点击无效的情况。




