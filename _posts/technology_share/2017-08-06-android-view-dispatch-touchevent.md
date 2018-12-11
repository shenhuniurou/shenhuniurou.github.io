---
layout: post
title: View的事件分发机制
category: blog
tags: Android、View
---


事件分发机制在View体系中几乎是最重要的知识点，理解透彻后在以后自定义各种复杂的View时我们就能更加得心应手了。


#### 点击事件的传递规则

点击事件即MotionEvent，点击事件的事件分发，也就是对MotionEvent事件的分发过程，当一个MotionEvent产生之后，系统需要把这个事件传递到一个具体的View，而这个传递过程就是分发过程。点击事件的分发过程由下面三个很重要的方法共同完成，`dispatchTouchEvent`、`onInterceptTouchEvent`、`onTouchEvent`。

- dispatchTouchEvent：用来进行事件的分发，如果事件能传递给当前View，那么此方法一定会被调用，返回结果受当前View的onTouchEvent和下级View的dispatchTouchEvent方法影响，表示是否消耗当前事件。

- onInterceptTouchEvent：该方法在dispatchTouchEvent方法内部调用，用来判断是否拦截某个事件，如果当前View拦截了某个事件，那么在同一个事件序列中，此方法不会被再次调用，返回的结果表示是否拦截当前事件。

- onTouchEvent：该方法在dispatchTouchEvent方法内部调用，用来处理点击事件，返回结果表示是否消耗当前事件，如果不消耗，则在同一事件序列中，当前View无法再次接收到事件。

**这个三个方法的关系：**

```
public boolean dispatchTouchEvent(MotionEvent event) {
    boolean consume = false;// 是否消耗事件
    if (onInterceptTouchEvent(event)) {
        consume = onTouchEvent(event);
    } else {
        consume = child.dispatchTouchEvent(event);
    }
    return consume;
}
```

上述伪代码表示的传递规则为：对于一个ViewGroup来说，点击事件产生后，首先会传递给它本身，这是它的dispatchTouchEvent方法会被调用，如果这个ViewGroup的onInterceptTouchEvent方法返回的是true就表示它要拦截这个事件，接着，这个事件就会交给这个ViewGroup来处理了，接着它的onTouchEvent方法就会被调用；但是如果这个ViewGroup的onInterceptTouchEvent返回false就表示它不拦截当前事件，这时事件就会继续传递给它的子元素，接着子元素的dispatchTouchEvent方法会被调用，过程反复直到事件被最终处理。

当一个View需要处理事件时，如果它设置了onTouchListener，那么onTouchListener中的onTouch方法会被回调。这时，事件如何处理还要看onTouch的返回值，如果返回false，则当前View的onTouchEvent方法会被调用，如果返回true，那么onTouchEvent方法不会被调用。因此，给View设置onTouchListener，其优先级比onTouchEvent更高，在onTouchEvent方法中，如果当前View有设置OnClickListener，那么它的onClick方法将会被调用，可以看出，OnClickListener对View的优先级最低，处于事件传递末端。

当一个点击事件产生后，它的传递过程遵循以下的顺序：Activity->Window->View，即事件总是先传递给Activity，Activity再传给Window，Window再传给最顶层的View，顶层View接收到事件后，就会按照事件分发机制去分发事件。如果一个View的onTouchEvent方法中返回了false，那么它的父容器的onTouchEvent将会被调用，依此类推。如果所有的元素都不处理这个事件，那么这个事件最终将会传递给Activity处理，即Activity的onTouchEvent方法将会被调用。


**关于事件传递机制的一些结论**

- 同一个事件序列是指从手指接触屏幕的那一刻起，到手指离开屏幕的那一刻结束，在这个过程中所产生的一系列事件，这个事件序列以DOWN事件开始，以UP事件结束，中间有N多个MOVE事件。

- 正常情况下，一个事件序列只能被一个View拦截且消耗。因为一旦一个元素拦截了某次事件，那么同一个事件序列内的所有事件都会直接交给它处理，因此同一个事件序列中的事件不能分别由两个View同时处理。

- 某个View一旦决定拦截，那么这一个事件序列都只能由它来处理，并且它的onInterceptTouchEvent方法不会再被调用，就是说当一个View决定拦截一个事件后，那么系统会把同一个事件序列内的其他方法都直接交给它来处理，因此就不用再调用这个View的onInterceptTouchEvent方法去询问它是否要拦截了。

- 某个View一旦开始处理事件，如果它不消耗ACTION_DOWN（onTouchEvent方法返回了false），那么同一事件序列中的其他事件也不会再交给它来处理，并且事件将重新交给它的父元素去处理，也就是父元素的onTouchEvent方法会被调用。

- 如果View不消耗除ACTION_DOWN以外的其他事件，那么这个点击事件会消失，此时父元素的onTouchEvent方法并不会被调用，并且当前View可以持续收到手续的事件，最终这些消失的点击事件会传递给Activity处理。

- ViewGroup默认不拦截任何事件。因为在Android源码中ViewGroup的onInterceptTouchEvent方法默认返回false。

- View没有onInterceptTouchEvent方法，一旦有点击事件传递给它，那么它的onTouchEvent方法就会被调用。

- View的onTouchEvent方法默认都会返回true表示消耗事件，除非它是不可点击的（clickable和longClickable同时为false）。View的longClickable属性默认为false，clickable属性要分情况，比如Button的clickable默认为true，而TextView的clickable默认为false。

- View的enable属性不影响onTouchEvent的默认返回值。即使一个View是disable状态的，只要它的clickable或者longClickable有一个为true，那么它的onTouchEvent就返回true。

- onClick方法会被调用的前提是当前View是可点击的，并且它收到了ACTION_DOWN和ACTION_UP事件。

- 事件传递的过程是由外向内的，即事件总是先传给父元素，然后由父元素分发给子View，通过requestDisallowInterceptTouchEvent方法可以在子元素中干预父元素的事件分发过程，但是ACTION_DOWN事件除外。


#### 事件分发的源码解析

当一个点击事件产生时，最先传递给当前的Activity，由Activity的dispatchTouchEvent来进行事件分发，具体是由Activity内部的Window来完成的。Window会将事件传递给decor view，decor view一般就是当前界面的底层容器（也就是setContentView所设置的View的父容器），通过Activity.getWindow.getDecorView()可以获得。

下面是Activity的dispatchTouchEvent方法：

```
public boolean dispatchTouchEvent(MotionEvent ev) {
    if (ev.getAction() == MotionEvent.ACTION_DOWN) {
        onUserInteraction();
    }
    if (getWindow().superDispatchTouchEvent(ev)) {
        return true;
    }
    return onTouchEvent(ev);
}
```

首先事件开始交给Activity所属的Window进行分发，如果返回true，整个事件循环就结束了，返回false表示事件没人处理，所有View的onTouchEvent都返回了false，那么Activity的onTouchEvent方法会被调用。

接着看Window如何将事件传递给ViewGroup。我们知道Window其实是一个抽象类，它的实现类其实是`PhoneWindow`，我们来看看PhoneWindow中的superDispatchTouchEvent方法，这个类的路径是/Users/daniel/Library/Android/sdk/sources/android-xx/com/android/internal/policy/PhoneWindow.java。

```
@Override
public boolean superDispatchTouchEvent(MotionEvent event) {
    return mDecor.superDispatchTouchEvent(event);
}
```

PhoneWindow直接将事件传递给了DecorView，这个DecorView的类所在目录和PhoneWindow的路径一样。

```
public class DecorView extends FrameLayout implements RootViewSurfaceTaker, WindowCallbacks {}
```

在PhoneWindow中是这样获取DecorView的：

```
@Override
public final View getDecorView() {
    if (mDecor == null || mForceDecorInstall) {
        installDecor();
    }
    return mDecor;
}
```

我们通过

```
((ViewGroup) getWindow().getDecorView().findViewById(android.R.id.content)).getChildAt(0)
```

这种方式可以获取到当前Activity所设置的View，这个mDecor就是getWindow().getDecorView()返回的View，而通过setContentView设置的View就是它的一个子View。目前事件已经传递到了DecorView，而DecorView又是继承自FrameLayout的，所以事件之后会传递到DecorView的子View也就是Activity里面通过setContentView设置的View，这个View是根View，也叫顶级View，一般来说，根View都是ViewGroup。

ViewGroup对点击事件的分发过程，主要实现在ViewGroup的dispatchTouchEvent方法中，它描述的是当前View是否拦截点击事件的逻辑。先看ViewGroup的dispatchTouchEvent方法中这么一段代码：

```
// Check for interception.
final boolean intercepted;
if (actionMasked == MotionEvent.ACTION_DOWN
        || mFirstTouchTarget != null) {
    final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
    if (!disallowIntercept) {
        intercepted = onInterceptTouchEvent(ev);
        ev.setAction(action); // restore action in case it was changed
    } else {
        intercepted = false;
    }
} else {
    // There are no touch targets and this action is not an initial down
    // so this view group continues to intercept touches.
    intercepted = true;
}
```

可以看出这一段是检查拦截的逻辑。ViewGroup会在下面两种情况下判断是否要拦截事件，一是事件类型为ACTION_DOWN，二是mFirstTouchTarget不等于null，这个mFirstTouchTarget在事件被ViewGroup的子元素成功处理时它会被赋值并指向子元素，也就是说当ViewGroup不拦截事件并将事件交给子元素处理时mFirstTouchTarget就不等于null，反过来，如果ViewGroup被拦截了，那么mFirstTouchTarget就等于null，所以当ACTION_MOVE和ACTION_UP事件到来时由于
```
(actionMasked == MotionEvent.ACTION_DOWN || mFirstTouchTarget != null) 
```
这个条件不满足，所以不会再调用onInterceptTouchEvent方法，并且同意事件序列中的其它事件都会默认交给ViewGroup来处理。



有一种特殊情况，就是FLAG_DISALLOW_INTERCEPT这个标记位被设置后，ViewGroup将无法拦截除了ACTION_DOWN之外的其它所有点击事件，这个标记位是通过requestDisallowInterceptTouchEvent方法来设置的，一般用在子View 中。为什么说是除了ACTION_DOWN之外的其它所有点击事件都会拦截，而不拦截ACTION_DOWN事件呢？因为ViewGroup在分发事件时，如果是ACTION_DOWN就会重置FLAG_DISALLOW_INTERCEPT这个标记位，导致子View中设置的这个标记位无效。因此，当面对ACTION_DOWN事件时，ViewGroup都是调用自己的onInterceptTouchEvent方法来询问自己是否要拦截事件。


```
// Handle an initial down.
if (actionMasked == MotionEvent.ACTION_DOWN) {
    // Throw away all previous state when starting a new touch gesture.
    // The framework may have dropped the up or cancel event for the previous gesture
    // due to an app switch, ANR, or some other state change.
    cancelAndClearTouchTargets(ev);
    resetTouchState();
}

private void resetTouchState() {
    clearTouchTargets();
    resetCancelNextUpFlag(this);
    mGroupFlags &= ~FLAG_DISALLOW_INTERCEPT;
    mNestedScrollAxes = SCROLL_AXIS_NONE;
}
```

上面这段代码就是当ACTION_DOWN事件到来时，ViewGroup重置FLAG_DISALLOW_INTERCEPT标记位的。

**总结**

当ViewGroup决定拦截事件后，后续的事件都会默认交给它处理并不再调用它的onInterceptTouchEvent方法。FLAG_DISALLOW_INTERCEPT这个标记位的作用就是让ViewGroup不拦截事件。onInterceptTouchEvent方法不是每次都会调动，如果我们想提前处理所有的点击事件，就要选择dispatchTouchEvent方法，因为这个方法是每次都会调用的。

当ViewGroup不再拦截的时候，事件会向下分发交给它的子View进行处理。

```
final View[] children = mChildren;
for (int i = childrenCount - 1; i >= 0; i--) {
    final int childIndex = getAndVerifyPreorderedIndex(
            childrenCount, i, customOrder);
    final View child = getAndVerifyPreorderedView(
            preorderedList, children, childIndex);

    // If there is a view that has accessibility focus we want it
    // to get the event first and if not handled we will perform a
    // normal dispatch. We may do a double iteration but this is
    // safer given the timeframe.
    if (childWithAccessibilityFocus != null) {
        if (childWithAccessibilityFocus != child) {
            continue;
        }
        childWithAccessibilityFocus = null;
        i = childrenCount - 1;
    }

    if (!canViewReceivePointerEvents(child)
            || !isTransformedTouchPointInView(x, y, child, null)) {
        ev.setTargetAccessibilityFocus(false);
        continue;
    }

    newTouchTarget = getTouchTarget(child);
    if (newTouchTarget != null) {
        // Child is already receiving touch within its bounds.
        // Give it the new pointer in addition to the ones it is handling.
        newTouchTarget.pointerIdBits |= idBitsToAssign;
        break;
    }

    resetCancelNextUpFlag(child);
    if (dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)) {
        // Child wants to receive touch within its bounds.
        mLastTouchDownTime = ev.getDownTime();
        if (preorderedList != null) {
            // childIndex points into presorted list, find original index
            for (int j = 0; j < childrenCount; j++) {
                if (children[childIndex] == mChildren[j]) {
                    mLastTouchDownIndex = j;
                    break;
                }
            }
        } else {
            mLastTouchDownIndex = childIndex;
        }
        mLastTouchDownX = ev.getX();
        mLastTouchDownY = ev.getY();
        newTouchTarget = addTouchTarget(child, idBitsToAssign);
        alreadyDispatchedToNewTouchTarget = true;
        break;
    }

    // The accessibility focus didn't handle the event, so clear
    // the flag and do a normal dispatch to all children.
    ev.setTargetAccessibilityFocus(false);
}
```

这段代码就是遍历ViewGroup的子元素，然后判断子元素是否能够接收到点击事件。是否能够接收到点击事件由两个因素决定，看下面的代码：

```
private static boolean canViewReceivePointerEvents(@NonNull View child) {
    return (child.mViewFlags & VISIBILITY_MASK) == VISIBLE || child.getAnimation() != null;
}
```

一个因素就是子元素是否在播放动画，另一个因素是点击时间的坐标是否落在子元素的区域内。如果满足这两个条件，那么事件就会传递给它来处理。可以看到dispatchTransformedTouchEvent方法实际上调用的就是子元素的dispatchTouchEvent方法。

```
if (cancel || oldAction == MotionEvent.ACTION_CANCEL) {
    event.setAction(MotionEvent.ACTION_CANCEL);
    if (child == null) {
        handled = super.dispatchTouchEvent(event);
    } else {
        handled = child.dispatchTouchEvent(event);
    }
    event.setAction(oldAction);
    return handled;
}
```

也就是说如果子元素的dispatchTouchEvent方法返回true，那么break循环，并且给mFirstTouchTarget赋值。如果返回false，那么会继续循环直至找到能处理事件的子元素。如果遍历了所有子元素事件都没有被处理，要么ViewGroup没有子元素要么是子元素处理了点击事件，但在dispatchTouchEvent方法中返回了false，这一般是因为子元素在onTouchEvent方法中返回了false。这两种情况下ViewGroup会自己处理点击事件。

```
/**
 * Adds a touch target for specified child to the beginning of the list.
 * Assumes the target child is not already present.
 */
private TouchTarget addTouchTarget(@NonNull View child, int pointerIdBits) {
    final TouchTarget target = TouchTarget.obtain(child, pointerIdBits);
    target.next = mFirstTouchTarget;
    mFirstTouchTarget = target;
    return target;
}
```

当mFirstTouchTarget为null时，ViewGroup会调用super.dispatchTouchEvent方法，因为下面的代码中第三个参数child它传的是null，而child为null时，正是调用的super.dispatchTouchEvent方法，我们知道ViewGroup也是继承View，所以事件将由View来处理。

```
if (mFirstTouchTarget == null) {
    // No touch targets so treat this as an ordinary view.
    handled = dispatchTransformedTouchEvent(ev, canceled, null, TouchTarget.ALL_POINTER_IDS);
} 
```



View的点击事件的处理比较简单。

```
public boolean dispatchTouchEvent(MotionEvent event) {
    // If the event should be handled by accessibility focus first.
    if (event.isTargetAccessibilityFocus()) {
        // We don't have focus or no virtual descendant has it, do not handle the event.
        if (!isAccessibilityFocusedViewOrHost()) {
            return false;
        }
        // We have focus and got the event, then use normal event dispatch.
        event.setTargetAccessibilityFocus(false);
    }

    boolean result = false;

    if (mInputEventConsistencyVerifier != null) {
        mInputEventConsistencyVerifier.onTouchEvent(event, 0);
    }

    final int actionMasked = event.getActionMasked();
    if (actionMasked == MotionEvent.ACTION_DOWN) {
        // Defensive cleanup for new gesture
        stopNestedScroll();
    }

    if (onFilterTouchEventForSecurity(event)) {
        if ((mViewFlags & ENABLED_MASK) == ENABLED && handleScrollBarDragging(event)) {
            result = true;
        }
        //noinspection SimplifiableIfStatement
        ListenerInfo li = mListenerInfo;
        if (li != null && li.mOnTouchListener != null
                && (mViewFlags & ENABLED_MASK) == ENABLED
                && li.mOnTouchListener.onTouch(this, event)) {
            result = true;
        }

        if (!result && onTouchEvent(event)) {
            result = true;
        }
    }

    if (!result && mInputEventConsistencyVerifier != null) {
        mInputEventConsistencyVerifier.onUnhandledEvent(event, 0);
    }

    // Clean up after nested scrolls if this is the end of a gesture;
    // also cancel it if we tried an ACTION_DOWN but we didn't want the rest
    // of the gesture.
    if (actionMasked == MotionEvent.ACTION_UP ||
            actionMasked == MotionEvent.ACTION_CANCEL ||
            (actionMasked == MotionEvent.ACTION_DOWN && !result)) {
        stopNestedScroll();
    }

    return result;
}
```

这里的View是不包括ViewGroup的，是单个元素，不会再有子元素了，无法传递事件，所以只需考虑它本身对事件的处理了。

首先View会判断有没有设置onTouchListener，如果onTouchListener中的onTouch方法返回true，那么表示onTouch方法将事件消耗掉了，onTouchEvent方法不会再被调用。

> 以前做ListView的item点击时，一般会有onItemClick处理单击事件和onItemLongClick处理长按事件，而onItemLongClick返回类型就是boolean类型，这个结果表示是否要消耗掉点击事件，如果返回true，那么在onItemClick中就不会再响应了，因为onItemLongClick比onItemClick的优先级高。所以要能同时响应这两个事件，onItemLongClick结果必须返回false。相信以前有人遇到过这个问题吧。

在View的onTouchEvent方法中对点击事件的处理：

```
if (((viewFlags & CLICKABLE) == CLICKABLE ||
        (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE) ||
        (viewFlags & CONTEXT_CLICKABLE) == CONTEXT_CLICKABLE) {
    switch (action) {
        case MotionEvent.ACTION_UP:
            boolean prepressed = (mPrivateFlags & PFLAG_PREPRESSED) != 0;
            if ((mPrivateFlags & PFLAG_PRESSED) != 0 || prepressed) {
                // take focus if we don't have it already and we should in
                // touch mode.
                boolean focusTaken = false;
                if (isFocusable() && isFocusableInTouchMode() && !isFocused()) {
                    focusTaken = requestFocus();
                }

                if (prepressed) {
                    // The button is being released before we actually
                    // showed it as pressed.  Make it show the pressed
                    // state now (before scheduling the click) to ensure
                    // the user sees it.
                    setPressed(true, x, y);
               }

                if (!mHasPerformedLongPress && !mIgnoreNextUpEvent) {
                    // This is a tap, so remove the longpress check
                    removeLongPressCallback();

                    // Only perform take click actions if we were in the pressed state
                    if (!focusTaken) {
                        // Use a Runnable and post this rather than calling
                        // performClick directly. This lets other visual state
                        // of the view update before click actions start.
                        if (mPerformClick == null) {
                            mPerformClick = new PerformClick();
                        }
                        if (!post(mPerformClick)) {
                            performClick();
                        }
                    }
                }

                if (mUnsetPressedState == null) {
                    mUnsetPressedState = new UnsetPressedState();
                }

                if (prepressed) {
                    postDelayed(mUnsetPressedState,
                            ViewConfiguration.getPressedStateDuration());
                } else if (!post(mUnsetPressedState)) {
                    // If the post failed, unpress right now
                    mUnsetPressedState.run();
                }

                removeTapCallback();
            }
            mIgnoreNextUpEvent = false;
            break;

        case MotionEvent.ACTION_DOWN:
            mHasPerformedLongPress = false;

            if (performButtonActionOnTouchDown(event)) {
                break;
            }

            // Walk up the hierarchy to determine if we're inside a scrolling container.
            boolean isInScrollingContainer = isInScrollingContainer();

            // For views inside a scrolling container, delay the pressed feedback for
            // a short period in case this is a scroll.
            if (isInScrollingContainer) {
                mPrivateFlags |= PFLAG_PREPRESSED;
                if (mPendingCheckForTap == null) {
                    mPendingCheckForTap = new CheckForTap();
                }
                mPendingCheckForTap.x = event.getX();
                mPendingCheckForTap.y = event.getY();
                postDelayed(mPendingCheckForTap, ViewConfiguration.getTapTimeout());
            } else {
                // Not inside a scrolling container, so show the feedback right away
                setPressed(true, x, y);
                checkForLongClick(0, x, y);
            }
            break;

        case MotionEvent.ACTION_CANCEL:
            setPressed(false);
            removeTapCallback();
            removeLongPressCallback();
            mInContextButtonPress = false;
            mHasPerformedLongPress = false;
            mIgnoreNextUpEvent = false;
            break;

        case MotionEvent.ACTION_MOVE:
            drawableHotspotChanged(x, y);

            // Be lenient about moving outside of buttons
            if (!pointInView(x, y, mTouchSlop)) {
                // Outside button
                removeTapCallback();
                if ((mPrivateFlags & PFLAG_PRESSED) != 0) {
                    // Remove any future long press/tap checks
                    removeLongPressCallback();

                    setPressed(false);
                }
            }
            break;
    }

    return true;
}
```

简单来说就是CLICKABLE和LONG_CLICKABLE有一个为true那么这个View就可以消耗这个事件，最后onTouchEvent返回true。在ACTION_UP事件中最后会执行performClick方法，在其内部会调用listener的onClick方法：

```
public boolean performClick() {
    final boolean result;
    final ListenerInfo li = mListenerInfo;
    if (li != null && li.mOnClickListener != null) {
        playSoundEffect(SoundEffectConstants.CLICK);
        li.mOnClickListener.onClick(this);
        result = true;
    } else {
        result = false;
    }

    sendAccessibilityEvent(AccessibilityEvent.TYPE_VIEW_CLICKED);
    return result;
}
```

View的LONG_CLICKABLE默认是false，CLICKABLE则是不同的View可能不同。比如Button默认是true，TextView默认是false。当通过setOnClickListener/setOnLongClickListener方法来设置点击监听时，就已经将LONG_CLICKABLE或者CLICKABLE属性设置为true了。

```
public void setOnLongClickListener(@Nullable OnLongClickListener l) {
    if (!isLongClickable()) {
        setLongClickable(true);
    }
    getListenerInfo().mOnLongClickListener = l;
}

public void setOnClickListener(@Nullable OnClickListener l) {
    if (!isClickable()) {
        setClickable(true);
    }
    getListenerInfo().mOnClickListener = l;
}
```

