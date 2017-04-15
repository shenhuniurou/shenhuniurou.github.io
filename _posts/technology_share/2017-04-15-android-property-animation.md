---
layout: post
title: Android动画之属性动画
category: 技术分享
tags: 属性动画
---



属性动画和补间动画的不同之处就是它通过动态改变对象的属性从而达到动画效果，它可以对任何对象做动画，并且动画效果也得到了加强，因为它不再只是有像补间动画平移、缩放、旋转和渐变这四种简单的变换了，只要对象有某个属性，它都能实现该属性在一段时间内变化的动画效果，但它是从API11才开始有的，如果要兼容API11以前的版本，可以采用开元动画库[nineoldandroids](http://nineoldandroids.com)。

属性动画除了通过代码的方式实现，还可以通过xml来定义。代码的方式中比较常用的几个动画类：ValueAnimator、ObjectAnimator和AnimatorSet，AnimatorSet是动画集合，可以把多个动画放在AnimatorSet中，同时执行。下面根据几个具体的情形简单使用者几个类做一些动画效果。

## 代码方式实现

**平移动画**

改变一个对象的translationX属性，使其在X轴上平移一段距离：

```java
TextView textview = (TextView) findViewById(R.id.textview);
textview.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        ObjectAnimator.ofFloat(v, "translationX", -200).start();
    }
});
```

效果如图：

![平移动画](https://static.oschina.net/uploads/img/201704/13232023_PFfb.gif "平移动画")

**渐变动画**

```java
TextView textview = (TextView) findViewById(R.id.textview);
textview.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        ValueAnimator colorAnim = ObjectAnimator.ofInt(v, "backgroundColor", getColor(R.color.colorPrimary), getColor(R.color.colorAccent));
        colorAnim.setDuration(2000); // 动画时间为2s
        colorAnim.setEvaluator(new ArgbEvaluator()); // 设置估值器
        colorAnim.setRepeatCount(ValueAnimator.INFINITE); // 设置动画无限重复执行
        colorAnim.setRepeatMode(ValueAnimator.REVERSE); // 设置变化反转效果，即第一次动画执行完后再次执行时背景色时从后面的颜色值往前面的变化
        colorAnim.start();
    }
});
```

效果如图：

![渐变动画](https://static.oschina.net/uploads/img/201704/13233858_wk7n.gif "渐变动画")

**动画集合AnimatorSet**

```java
TextView textview = (TextView) findViewById(R.id.textview);

textview.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {

        AnimatorSet animatorSet = new AnimatorSet();

        animatorSet.playTogether(
                ObjectAnimator.ofFloat(v, "rotation", 0, 360, 0),
                ObjectAnimator.ofFloat(v, "scaleX", 1, 2.5f, 1),
                ObjectAnimator.ofFloat(v, "scaleY", 1, 2.5f, 1),
                ObjectAnimator.ofFloat(v, "alpha", 1, 0.3f, 1)
        );

        animatorSet.setDuration(5000);
        animatorSet.start();

    }
});
```

效果如图：

![动画集合](http://upload-images.jianshu.io/upload_images/1159224-6552fb76542d1f6c.gif?imageMogr2/auto-orient/strip)



## xml定义方式实现

首先我们需要定义一个动画的xml文件，属性动画需要定义在res/animator/目录下：

```java
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android"

    android:ordering="together">

    <objectAnimator
        android:duration="2000"
        android:propertyName="scaleX"
        android:repeatCount="2"
        android:repeatMode="reverse"
        android:valueFrom="1"
        android:valueTo="2.5f"
        android:valueType="floatType" />

    <animator
        android:duration="2000"
        android:propertyName="scaleX"
        android:repeatCount="2"
        android:repeatMode="reverse"
        android:startOffset="1000"
        android:valueFrom="1"
        android:valueTo="2.5f"
        android:valueType="floatType" />

</set>
```

其中的标签和代码中的属性都是对应的，很好理解，在xml中可以定义`ValueAnimator`、`ObjectAnimator`和`AnimatorSet
`;
<set>标签对应的AnimatorSet，<animator>对应ValueAnimator，<objectAnimator>对应ObjectAnimator;
<set>中的属性android:ordering表示它里面的动画集合执行的方式，together表示多个子动画同时执行，sequentially则表示按照xml中的先后顺序依次执行，默认是together；

android:propertyName表示动画作用于哪个属性的名称，比如translationX，rorationY等，需要注意的是，<animator>标签是没有这个属性的，其余属性和objectAnimator相同；

android:duration表示动画时长；

android:valueFrom表示属性的起始值；

android:valueTo表示属性的最终值;

android:startOffset表示动画的延迟时间，动画开始后，延迟多长时间才开始真正执行；

android:repeatCount表示动画执行次数，默认是0，表示不重复执行，-1表示无限循环

android:repeatMode表示动画执行模式，它的值有两种，`reverse`表示反转重复执行，`restart`表示重新开始连续执行；

android:valueType表示android:propertyName指定的属性的类型，它的值有四种，`intType`、`floatType`、`colorType`和`pathType`，当指定的属性表示的是颜色时，使用colorType。

定义完xml后，我们需要将它作用到对应上来：

```java
TextView textview = (TextView) findViewById(R.id.textview);

textview.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
    
        AnimatorSet animatorSet = (AnimatorSet) AnimatorInflater.loadAnimator(MainActivity.this, R.anim.animator_textview);

        animatorSet.setTarget(v);
        animatorSet.start();
    }
});
```

效果如图：

![xml定义属性动画](https://static.oschina.net/uploads/img/201704/14004817_B2WS.gif "xml定义属性动画")

## 两种方式的利弊

通过xml定义的方式需要提前知道属性的起始值和结束值，这就决定了它的局限性，而通过代码的方式就更加灵活，我们可以在对象完全加载后获取其属性值，从而达到动态创建属性动画的效果。


## 插值器和估值器

TimeInterpolator即插值器，它的作用是根据时间消耗的百分比来计算出当前属性改变的百分比，Android系统自带的插值器有LinearInterpolator、AccelerateInterpolator、AccelerateDecelerateInterpolator和DecelerateInterpolator等，LinearInterpolator是线性插值器，是匀速动画，AccelerateInterpolator是加速插值器，动画越来越快，AccelerateDecelerateInterpolator是加速减速插值器，两头慢中间快，DecelerateInterpolator是减速插值器，动画越来越慢。

TypeEvaluator即估值器，它的作用是根据当前属性改变的百分比来计算改变后的属性值，系统自带的估值器有IntEvaluator、FloatEvaluator和ArgbEvaluator，主要是针对不同属性类型的估计计算。

插值器和估值器能让动画按照特定的速率来进行改变。

我们来看看`LinearInterpolator`的源码：

```java
public class LinearInterpolator extends BaseInterpolator implements NativeInterpolatorFactory {

    public LinearInterpolator() {
    }

    public LinearInterpolator(Context context, AttributeSet attrs) {
    }

    public float getInterpolation(float input) {
        return input;
    }

    /** @hide */
    @Override
    public long createNativeInterpolator() {
        return NativeInterpolatorFactoryHelper.createLinearInterpolator();
    }
}
```

再来看看`FloatEvaluator`的源码：

```java
public class FloatEvaluator implements TypeEvaluator<Number> {

    public Float evaluate(float fraction, Number startValue, Number endValue) {
        float startFloat = startValue.floatValue();
        return startFloat + fraction * (endValue.floatValue() - startFloat);
    }
    
}
```

`BaseInterpolator`就是一个实现了`Interpolator`接口的抽象类，`getInterpolation`方法就是根据当前属性改变的百分比计算出一个估值小数，它的参数input即当前属性改变的百分比，`TypeEvaluator`是一个估值接口，如果需要自定义估值器，就必须实现该接口，并重写`evaluate`方法，它的三个参数分别表示估值小数、属性起始值和结束值。为什么使用线性插值器就表示是匀速动画？就是因为它的`getInterpolation`方法得到的估值小数就是当前属性改变的百分比，估值小数代入估值器中计算，任何一段时间内的属性的改变速率都相同。



插值器和估值器除了系统提供的以外，还可以自定义，插值器只需要实现`Interpolator`或者`TimeInterpolator`即可，自定义估值器需要实现`TypeEvaluator`接口，如果是要针对其他非int、float、color类型的属性做动画，我们就必须要自定义类型估值器。

**自定义插值器**

```java
package com.shenhuniurou.propertyanimation;

import android.view.animation.Interpolator;

/**
 * Created by shenhuniurou on 2017/4/15.
 */

public class CustomInterpolator implements Interpolator {

    private float mFactor = 1.0f;

    public CustomInterpolator() {
    }

    public CustomInterpolator(float mFactor) {
        this.mFactor = mFactor;
    }
    

    @Override
    public float getInterpolation(float input) {
        float result;
        if (mFactor == 1.0f) {
            result = input * input;
        } else {
            result = input * input * 2 * mFactor;
        }
        return result;
    }

}
```

**自定义估值器**

```java
package com.shenhuniurou.propertyanimation;

import android.animation.TypeEvaluator;

/**
 * Created by shenhuniurou on 2017/4/15.
 */

public class CustomEvaluator implements TypeEvaluator<Float> {

    @Override
    public Float evaluate(float fraction, Float startValue, Float endValue) {
        return fraction * endValue + startValue;
    }

}
```

> 如果自定义的估值器是作用于对象的非int、float和Color属性，那么这个对象必须要有该属性，并且有getter和setter方法。

## 监听属性动画

属性动画的监听有两个接口：`AnimatorListener`和`AnimatorUpdateListener`，AnimatorUpdateListener的定义如下：

```java
public static interface AnimatorUpdateListener {
    
    void onAnimationUpdate(ValueAnimator animation);

}
```

用法：

```java
colorAnim.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
    @Override
    public void onAnimationUpdate(ValueAnimator animation) {

    }
});
```

它会监听动画的整个过程，由于动画是由许多帧组成的，每播放一帧，`onAnimationUpdate`方法就会被调用一次。

AnimatorListener的定义如下：

```java
public static interface AnimatorListener {
        
    void onAnimationStart(Animator animation);

    void onAnimationEnd(Animator animation);
    
    void onAnimationCancel(Animator animation);
    
    void onAnimationRepeat(Animator animation);
}
```

用法：

```java
colorAnim.addListener(new Animator.AnimatorListener() {
    @Override
    public void onAnimationStart(Animator animation) {

    }

    @Override
    public void onAnimationEnd(Animator animation) {

    }

    @Override
    public void onAnimationCancel(Animator animation) {

    }

    @Override
    public void onAnimationRepeat(Animator animation) {

    }
});
```

这四个方法分别在动画的开始、结束、取消和重复播放时调用，另外系统还提供了AnimatorListenerAdapter这个类：

```java
colorAnim.addListener(new AnimatorListenerAdapter() {
    @Override
    public void onAnimationEnd(Animator animation) {
        super.onAnimationEnd(animation);
    }

    @Override
    public void onAnimationStart(Animator animation) {
        super.onAnimationStart(animation);
    }
});
```

使用这个类和AnimatorListener不同，这里我们可以选择要实现的方法，而不是重写所有的方法。

## 属性动画的工作原理

属性动画要求动画作用的对象必须提供这个属性的setter和getter方法，确切的说是setter方法，因为属性动画是根据该属性的初始值和结束值，动态去调用该属性的set方法，每播放一帧，就会调用一次，也就是说，如果我们要对某个对象的某个属性做动画，就必须提供setter方法，如果开始动画时没有传递初始值，那么还需要提供getter方法，因为系统要去获取到初始值，没有没有提供getter方法，程序会Crash。另外，如果要想属性的改变产生动态效果，那么这种改变就必须在执行setter方法后通过肉眼可见的变化反映出来，否则动画是没有效果的。

**那么如何解决这种情况的问题呢**

- 对原始对象进行包装，为包装类提供getter和setter方法。

```java
WrapperClass wrapper = new WrapperClass(targetView);
ObjectAnimator.ofFloat(wrapper, propertyName, endValue).setDuration(duration).start();

private class WrapperClass {

    private View mTarget;

    public WrapperClass(View target) {
        mTarget = target;
    }

    public void getter() {}

    public void setter() {}

}
```

- 使用ValueAnimator监听动画过程，自己实现属性的改变。

接着看属性动画是怎么工作的：我们知道要开始播放动画，需要调用方法start()。`ObjectAnimator`的start方法：

```java
@Override
public void start() {
    AnimationHandler.getInstance().autoCancelBasedOn(this);
    if (DBG) {
        Log.d(LOG_TAG, "Anim target, duration: " + getTarget() + ", " + getDuration());
        for (int i = 0; i < mValues.length; ++i) {
            PropertyValuesHolder pvh = mValues[i];
            Log.d(LOG_TAG, "   Values[" + i + "]: " +
                pvh.getPropertyName() + ", " + pvh.mKeyframes.getValue(0) + ", " +
                pvh.mKeyframes.getValue(1));
        }
    }
    super.start();
}
```

我们看到代码最后调用父类的start方法，而ObjectAnimator是继承自ValueAnimator的，再来看它的start方法：

```java
private void start(boolean playBackwards) {
    if (Looper.myLooper() == null) {
        throw new AndroidRuntimeException("Animators may only be run on Looper threads");
    }
    mReversing = playBackwards;
    if (playBackwards && mSeekFraction != -1 && mSeekFraction != 0) {
        if (mRepeatCount == INFINITE) {
            float fraction = (float) (mSeekFraction - Math.floor(mSeekFraction));
            mSeekFraction = 1 - fraction;
        } else {
            mSeekFraction = 1 + mRepeatCount - mSeekFraction;
        }
    }
    mStarted = true;
    mPaused = false;
    mRunning = false;
    mAnimationEndRequested = false;
    mLastFrameTime = 0;
    AnimationHandler animationHandler = AnimationHandler.getInstance();
    animationHandler.addAnimationFrameCallback(this, (long) (mStartDelay * sDurationScale));

    if (mStartDelay == 0 || mSeekFraction >= 0) {
        startAnimation();
        if (mSeekFraction == -1) {
            setCurrentPlayTime(0);
        } else {
            setCurrentFraction(mSeekFraction);
        }
    }
}
```

这里我们可以看出属性动画必须要运行在有Looper的线程中，否则会抛出运行时异常。上述这段代码最后我们看到ValueAnimator把当前动画处理交给了`AnimationHandler `这个类去处理，并注册了动画帧的回调监听。AnimationHandler是所有活动ValueAnimator共享，它能确保动画值的设置将发生在动画开始的同一个线程上，并且所有动画将共享相同的时间来计算它们的值。这个类会使用定时脉冲进行周期性的回调，从而进行UI的更新，也就是当一帧播放完毕后，会发送一个脉冲，回调ValueAnimator的`doAnimationFrame`方法。

```java
private final Choreographer.FrameCallback mFrameCallback = new Choreographer.FrameCallback() {
    @Override
    public void doFrame(long frameTimeNanos) {
        doAnimationFrame(getProvider().getFrameTime());
        if (mAnimationCallbacks.size() > 0) {
            getProvider().postFrameCallback(this);
        }
    }
};
```

AnimationHandler类中的doAnimationFrame方法：

```java
private void doAnimationFrame(long frameTime) {
    int size = mAnimationCallbacks.size();
    long currentTime = SystemClock.uptimeMillis();
    for (int i = 0; i < size; i++) {
        final AnimationFrameCallback callback = mAnimationCallbacks.get(i);
        if (callback == null) {
            continue;
        }
        if (isCallbackDue(callback, currentTime)) {
            callback.doAnimationFrame(frameTime);
            if (mCommitCallbacks.contains(callback)) {
                getProvider().postCommitCallback(new Runnable() {
                    @Override
                    public void run() {
                        commitAnimationFrame(callback, getProvider().getFrameTime());
                    }
                });
            }
        }
    }
    cleanUpList();
}
```

可以看到有回调ValueAnimator的doAnimationFrame方法：

```java
public final void doAnimationFrame(long frameTime) {
    AnimationHandler handler = AnimationHandler.getInstance();
    if (mLastFrameTime == 0) {
        // First frame
        handler.addOneShotCommitCallback(this);
        if (mStartDelay > 0) {
            startAnimation();
        }
        if (mSeekFraction < 0) {
            mStartTime = frameTime;
        } else {
            long seekTime = (long) (getScaledDuration() * mSeekFraction);
            mStartTime = frameTime - seekTime;
            mSeekFraction = -1;
        }
        mStartTimeCommitted = false; // allow start time to be compensated for jank
    }
    mLastFrameTime = frameTime;
    if (mPaused) {
        mPauseTime = frameTime;
        handler.removeCallback(this);
        return;
    } else if (mResumed) {
        mResumed = false;
        if (mPauseTime > 0) {
            // Offset by the duration that the animation was paused
            mStartTime += (frameTime - mPauseTime);
            mStartTimeCommitted = false; // allow start time to be compensated for jank
        }
        handler.addOneShotCommitCallback(this);
    }
    final long currentTime = Math.max(frameTime, mStartTime);
    boolean finished = animateBasedOnTime(currentTime);

    if (finished) {
        endAnimation();
    }
}
```

这段代码最后我们看到：

```java
 boolean finished = animateBasedOnTime(currentTime);
```

所以真正执行动画的应该就是方法`animateBasedOnTime`了，继续跟进去看:

```java
boolean animateBasedOnTime(long currentTime) {
    boolean done = false;
    if (mRunning) {
        final long scaledDuration = getScaledDuration();
        final float fraction = scaledDuration > 0 ?
                (float)(currentTime - mStartTime) / scaledDuration : 1f;
        final float lastFraction = mOverallFraction;
        final boolean newIteration = (int) fraction > (int) lastFraction;
        final boolean lastIterationFinished = (fraction >= mRepeatCount + 1) &&
                (mRepeatCount != INFINITE);
        if (scaledDuration == 0) {
            // 0 duration animator, ignore the repeat count and skip to the end
            done = true;
        } else if (newIteration && !lastIterationFinished) {
            // Time to repeat
            if (mListeners != null) {
                int numListeners = mListeners.size();
                for (int i = 0; i < numListeners; ++i) {
                    mListeners.get(i).onAnimationRepeat(this);
                }
            }
        } else if (lastIterationFinished) {
            done = true;
        }
        mOverallFraction = clampFraction(fraction);
        float currentIterationFraction = getCurrentIterationFraction(mOverallFraction);
        animateValue(currentIterationFraction);
    }
    return done;
}
```

代码最后调用了`animateValue`方法，根据方法名我们也大致猜出意思来了

```java
void animateValue(float fraction) {
    fraction = mInterpolator.getInterpolation(fraction);
    mCurrentFraction = fraction;
    int numValues = mValues.length;
    for (int i = 0; i < numValues; ++i) {
        mValues[i].calculateValue(fraction);
    }
    if (mUpdateListeners != null) {
        int numListeners = mUpdateListeners.size();
        for (int i = 0; i < numListeners; ++i) {
            mUpdateListeners.get(i).onAnimationUpdate(this);
        }
    }
}
```

上述代码中的`calculateValue`方法就是计算每帧动画所对应的属性值，但是这个仍然没有看到调用属性的setter和getter方法的部分。我们继续看ValueAnimator在初始化的时候，如果没有传初始值，getter方法会被调用。我们看到在初始化时，会调用`setValues`这个方法：

```java
public void setValues(PropertyValuesHolder... values) {
    int numValues = values.length;
    mValues = values;
    mValuesMap = new HashMap<String, PropertyValuesHolder>(numValues);
    for (int i = 0; i < numValues; ++i) {
        PropertyValuesHolder valuesHolder = values[i];
        mValuesMap.put(valuesHolder.getPropertyName(), valuesHolder);
    }
    // New property/values/target should cause re-initialization prior to starting
    mInitialized = false;
}
```

它是把每个属性值存放在一个value是`PropertyValuesHolder`的HashMap中，PropertyValuesHolder类中包含有关属性和动画期间该属性对用的值的信息。在它的`setupValue`方法中通过反射调用getter方法获取初始值并赋值。

```java
private void setupValue(Object target, Keyframe kf) {
    if (mProperty != null) {
        Object value = convertBack(mProperty.get(target));
        kf.setValue(value);
    } else {
        try {
            if (mGetter == null) {
                Class targetClass = target.getClass();
                setupGetter(targetClass);
                if (mGetter == null) {
                    // Already logged the error - just return to avoid NPE
                    return;
                }
            }
            Object value = convertBack(mGetter.invoke(target));
            kf.setValue(value);
        } catch (InvocationTargetException e) {
            Log.e("PropertyValuesHolder", e.toString());
        } catch (IllegalAccessException e) {
            Log.e("PropertyValuesHolder", e.toString());
        }
    }
}
```

而当动画开始后，需要调用set方法，也是在PropertyValuesHolder的`setAnimatedValue`方法中通过反射调用：

```java
void setAnimatedValue(Object target) {
    if (mProperty != null) {
        mProperty.set(target, getAnimatedValue());
    }
    if (mSetter != null) {
        try {
            mTmpValueArray[0] = getAnimatedValue();
            mSetter.invoke(target, mTmpValueArray);
        } catch (InvocationTargetException e) {
            Log.e("PropertyValuesHolder", e.toString());
        } catch (IllegalAccessException e) {
            Log.e("PropertyValuesHolder", e.toString());
        }
    }
}
```

> 以上源码的sdk版本是25。

## 需要注意的问题

如果我们在Activity中开启了一个无限循环的属性动画，那么在Activity退出时我们需要及时停止属性动画，否则会导致activity无法释放该动画的引用，从而造成内存泄漏的问题。


