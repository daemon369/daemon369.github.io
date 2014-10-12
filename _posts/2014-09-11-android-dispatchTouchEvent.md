---
layout: post
keywords: andrid, onInterceptTouchEvent, onTouchEvent, dispatchTouchEvent
description: onInterceptTouchEvent & onTouchEvent & dispatchTouchEvent
title: "Android Touch事件传递机制(三) -- Touch事件分发(dispatchTouchEvent)"
categories: [Android]
tags: [android, dispatchTouchEvent, OnTouchListener, onTouch, onTouchEvent, onInterceptTouchEvent]
group: Android
icon: file-alt
---
{% include JB/setup %}

***

(本文基于android-2.3.3_r1代码研究)

在[Android Touch事件传递机制(一)][1]和[Android Touch事件传递机制(二)][2]这两篇文章中研究了Android触屏事件的分发机制；本文从源码角度继续深入研究。

***

#一. ViewRoot

Android的所有触屏事件、按键事件、界面刷新等事件都是通过ViewRoot进行分发的。

ViewRoot实际就是一个Handler，接收native层传来的事件并进行分发。

下面看看ViewRoot对触屏事件处理的关键代码：

{% highlight java linenos %}
public final class ViewRoot extends Handler implements ViewParent,
        View.AttachInfo.Callbacks {

    @Override
    public void handleMessage(Message msg) {
        switch (msg.what) {

        case DISPATCH_POINTER: {
            MotionEvent event = (MotionEvent) msg.obj;
            try {
                deliverPointerEvent(event);
            } finally {
                event.recycle();
                if (msg.arg1 != 0) {
                    finishInputEvent();
                }

            }
        } break;

    }

}
{% endhighlight %}

ViewRoot接收到触屏事件，调用deliverPointerEvent方法。deliverPointerEvent方法关键代码如下：

<!--excerpt-->

{% highlight java linenos %}
private void deliverPointerEvent(MotionEvent event) {

    boolean handled;

    boolean isDown = event.getAction() == MotionEvent.ACTION_DOWN;

    handled = mView.dispatchTouchEvent(event);

    if (!handled && isDown) {

        final int edgeFlags = event.getEdgeFlags();

        if (edgeFlags != 0 && mView instanceof ViewGroup) {
            View nearest = FocusFinder.getInstance().findNearestTouchable(
                    ((ViewGroup) mView), x, y, direction, deltas);
            if (nearest != null) {

                mView.dispatchTouchEvent(event);
            }
        }
    }

}
{% endhighlight %}

可以看到最终事件会分发给mView的dispatchTouchEvent方法进行处理。mView是一个窗口关联的View树的最顶层View，通过ViewRoot的setView方法可以设置mView的值。

下面我们看下View和ViewGroup的dispatchTouchEvent方法。

***

#二. View.dispatchTouchEvent()

{% highlight java linenos %}
/**
 * Pass the touch screen motion event down to the target view, or this
 * view if it is the target.
 *
 * @param event The motion event to be dispatched.
 * @return True if the event was handled by the view, false otherwise.
 */
public boolean dispatchTouchEvent(MotionEvent event) {
    if (!onFilterTouchEventForSecurity(event)) {
        return false;
    }

    if (mOnTouchListener != null && (mViewFlags & ENABLED_MASK) == ENABLED &&
            mOnTouchListener.onTouch(this, event)) {
        return true;
    }
    return onTouchEvent(event);
}
{% endhighlight %}

View的dispatchTouchEvent方法处理逻辑很简单：

当View可点击且设置了OnTouchListener时，首先回调OnTouchListener接口的onTouch方法；

当onTouch方法返回true时，dispatchTouchEvent方法直接返回true；

当onTouch方法返回false时，dispatchTouchEvent方法再调用View的onTouchEvent方法，并返回onTouchEvent方法的返回值

这个逻辑对应了[文章2][2]中Demo的演示结果

***

#三. ViewGroup.dispatchTouchEvent()

ViewGroup中的dispatchTouchEvent方法就要复杂的多了，关键代码如下：

{% highlight java linenos %}
@Override
public boolean dispatchTouchEvent(MotionEvent ev) {

    final int action = ev.getAction();
    final float xf = ev.getX();
    final float yf = ev.getY();
    final float scrolledXFloat = xf + mScrollX;
    final float scrolledYFloat = yf + mScrollY;
    final Rect frame = mTempRect;

    boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;

    if (action == MotionEvent.ACTION_DOWN) {
        if (mMotionTarget != null) {
            mMotionTarget = null;
        }
        // If we're disallowing intercept or if we're allowing and we didn't
        // intercept
        if (disallowIntercept || !onInterceptTouchEvent(ev)) {
            // reset this event's action (just to protect ourselves)
            ev.setAction(MotionEvent.ACTION_DOWN);
            // We know we want to dispatch the event down, find a child
            // who can handle it, start with the front-most child.
            final int scrolledXInt = (int) scrolledXFloat;
            final int scrolledYInt = (int) scrolledYFloat;
            final View[] children = mChildren;
            final int count = mChildrenCount;

            for (int i = count - 1; i >= 0; i--) {
                final View child = children[i];
                if ((child.mViewFlags & VISIBILITY_MASK) == VISIBLE
                        || child.getAnimation() != null) {
                    child.getHitRect(frame);
                    if (frame.contains(scrolledXInt, scrolledYInt)) {
                        // offset the event to the view's coordinate system
                        final float xc = scrolledXFloat - child.mLeft;
                        final float yc = scrolledYFloat - child.mTop;
                        ev.setLocation(xc, yc);
                        child.mPrivateFlags &= ~CANCEL_NEXT_UP_EVENT;
                        if (child.dispatchTouchEvent(ev))  {
                            // Event handled, we have a target now.
                            mMotionTarget = child;
                            return true;
                        }
                        // The event didn't get handled, try the next view.
                        // Don't reset the event's location, it's not
                        // necessary here.
                    }
                }
            }
        }
    }

    boolean isUpOrCancel = (action == MotionEvent.ACTION_UP) ||
            (action == MotionEvent.ACTION_CANCEL);

    if (isUpOrCancel) {
        // Note, we've already copied the previous state to our local
        // variable, so this takes effect on the next event
        mGroupFlags &= ~FLAG_DISALLOW_INTERCEPT;
    }

    // The event wasn't an ACTION_DOWN, dispatch it to our target if
    // we have one.
    final View target = mMotionTarget;
    if (target == null) {
        // We don't have a target, this means we're handling the
        // event as a regular view.
        ev.setLocation(xf, yf);
        if ((mPrivateFlags & CANCEL_NEXT_UP_EVENT) != 0) {
            ev.setAction(MotionEvent.ACTION_CANCEL);
            mPrivateFlags &= ~CANCEL_NEXT_UP_EVENT;
        }
        return super.dispatchTouchEvent(ev);
    }

    // if have a target, see if we're allowed to and want to intercept its
    // events
    if (!disallowIntercept && onInterceptTouchEvent(ev)) {
        final float xc = scrolledXFloat - (float) target.mLeft;
        final float yc = scrolledYFloat - (float) target.mTop;
        mPrivateFlags &= ~CANCEL_NEXT_UP_EVENT;
        ev.setAction(MotionEvent.ACTION_CANCEL);
        ev.setLocation(xc, yc);
        if (!target.dispatchTouchEvent(ev)) {
            // target didn't handle ACTION_CANCEL. not much we can do
            // but they should have.
        }
        // clear the target
        mMotionTarget = null;
        // Don't dispatch this event to our own view, because we already
        // saw it when intercepting; we just want to give the following
        // event to the normal onTouchEvent().
        return true;
    }

    if (isUpOrCancel) {
        mMotionTarget = null;
    }

    // finally offset the event to the target's coordinate system and
    // dispatch the event.
    final float xc = scrolledXFloat - (float) target.mLeft;
    final float yc = scrolledYFloat - (float) target.mTop;
    ev.setLocation(xc, yc);

    if ((target.mPrivateFlags & CANCEL_NEXT_UP_EVENT) != 0) {
        ev.setAction(MotionEvent.ACTION_CANCEL);
        target.mPrivateFlags &= ~CANCEL_NEXT_UP_EVENT;
        mMotionTarget = null;
    }

    return target.dispatchTouchEvent(ev);
}
{% endhighlight %}

11行代码判断是否允许当前ViewGroup拦截touch事件；当允许拦截touch事件时，onInterceptTouchEvent方法才会起到拦截touch事件的作用；使用requestDisallowInterceptTouchEvent方法可以设置是否允许拦截touch事件

19行代码处，down事件在允许ViewGroup拦截touch事件时，先将down事件分发给ViewGroup的onInterceptTouchEvent方法处理；

当ViewGroup的onInterceptTouchEvent方法返回false时，再将down事件分发给子View；从最前方的子View到最底层的子View，依次调用子View的dispatchTouchEvent方法；若子View的dispatchEvent方法返回true，表示这个子View消费掉这次down事件，设置这个View为目标View(mMotionTarget)，并返回true，不再继续分发；若所有的子View都不消费掉这次down事件，则mMotionTarget为null，将该ViewGroup当做普通View，调用super.dispatchTouchEvent方法。

若事件不是down事件，或不允许该ViewGroup拦截touch事件，或ViewGroup的onInterceptTouchEvent方法返回了true，则从54行代码处执行。

66行代码处，mMotionTarget为null，表明没有子View消费掉down事件，则将该ViewGroup当做普通View，调用super.dispatchTouchEvent方法。

执行到79行代码处表明有子View消费了之前的down事件；后续的事件交由目标View处理；79行代码处，当允许ViewGroup拦截touch事件时，首先将后续事件分发到onInterceptTouchEvent方法中，如果onInterceptTouchEvent方法返回true，表示ViewGroup拦截后续事件，则mMotionTarget将接收到cancel事件，mMotionTarget同时被设置为null，这样后续事件不再分发给mMotionTarget；onInterceptTouchEvent返回false，事件继续传递；

97行代码处，接收到up事件或cancel事件，将事件分发给目标View，同时清除mMotionTarget，一次完整触屏操作就此结束。

***

#四. Activity.dispatchTouchEvent

前面分析了View以及ViewGroup对touch事件的分发处理机制，下面我们继续研究下Activity的dispatchTouchEvent方法。

Activity中提供了dispatchTouchEvent方法，这个方法又是什么时候被调用的呢？

我们知道在ViewRoot中touch首先分发给mView来处理的，那么mView到底是什么呢？

我们在创建一个Activity时，经常调用Activity的setContentView方法来显示自定义的布局文件：

{% highlight java linenos %}
/**
 * Set the activity content from a layout resource.  The resource will be
 * inflated, adding all top-level views to the activity.
 * 
 * @param layoutResID Resource ID to be inflated.
 */
public void setContentView(int layoutResID) {
    getWindow().setContentView(layoutResID);
}
{% endhighlight %}

这里getWindow获取的是PhoneWindow的实例，其setContentView如下：

{% highlight java linenos %}
@Override
public void setContentView(int layoutResID) {
    if (mContentParent == null) {
        installDecor();
    } else {
        mContentParent.removeAllViews();
    }
    mLayoutInflater.inflate(layoutResID, mContentParent);

    ......
}
{% endhighlight %}

可以看到传入的布局文件最终会展开并作为mContentParent的子View。那么mContentParent又是怎么产生的？看下installDecor方法：

{% highlight java linenos %}
private void installDecor() {
    if (mDecor == null) {
        mDecor = generateDecor();
        mDecor.setDescendantFocusability(ViewGroup.FOCUS_AFTER_DESCENDANTS);
        mDecor.setIsRootNamespace(true);
    }
    if (mContentParent == null) {
        mContentParent = generateLayout(mDecor);

        ......
    }
}
{% endhighlight %}

在installDecor方法中，创建了一个DecorView的实例并赋值给mDecor变量。然后在第8行调用generateLayout并传入mDecor变量：

{% highlight java linenos %}
protected ViewGroup generateLayout(DecorView decor) {
    // Apply data from current theme.

    ......

    // Inflate the window decor.

    int layoutResource;
    int features = getLocalFeatures();

    if ((features & ((1 << FEATURE_LEFT_ICON) | (1 << FEATURE_RIGHT_ICON))) != 0) {
        if (mIsFloating) {
            layoutResource = com.android.internal.R.layout.dialog_title_icons;
        } else {
            layoutResource = com.android.internal.R.layout.screen_title_icons;
        }
        // System.out.println("Title Icons!");
    } else if ((features & ((1 << FEATURE_PROGRESS) | (1 << FEATURE_INDETERMINATE_PROGRESS))) != 0) {
        // Special case for a window with only a progress bar (and title).
        // XXX Need to have a no-title version of embedded windows.
        layoutResource = com.android.internal.R.layout.screen_progress;
        // System.out.println("Progress!");
    } else if ((features & (1 << FEATURE_CUSTOM_TITLE)) != 0) {
        // Special case for a window with a custom title.
        // If the window is floating, we need a dialog layout
        if (mIsFloating) {
            layoutResource = com.android.internal.R.layout.dialog_custom_title;
        } else {
            layoutResource = com.android.internal.R.layout.screen_custom_title;
        }
    } else if ((features & (1 << FEATURE_NO_TITLE)) == 0) {
        // If no other features and not embedded, only need a title.
        // If the window is floating, we need a dialog layout
        if (mIsFloating) {
            layoutResource = com.android.internal.R.layout.dialog_title;
        } else {
            layoutResource = com.android.internal.R.layout.screen_title;
        }
        // System.out.println("Title!");
    } else {
        // Embedded, so no decoration is needed.
        layoutResource = com.android.internal.R.layout.screen_simple;
        // System.out.println("Simple!");
    }

    mDecor.startChanging();

    View in = mLayoutInflater.inflate(layoutResource, null);
    decor.addView(in, new ViewGroup.LayoutParams(MATCH_PARENT, MATCH_PARENT));

    ViewGroup contentParent = (ViewGroup)findViewById(ID_ANDROID_CONTENT);

    ......

    mDecor.finishChanging();

    return contentParent;
}
{% endhighlight %}

11行到44行是根据当前的主题(theme)来展开不同的布局文件，然后在48、49行将展开的布局添加到传入的mDecorView中。然后在51行找到mDecorView中id为Window.ID_ANDROID_CONTENT(com.android.internal.R.id.content)的View并返回，最终设置为mContentParent变量。

可以看出，Activity中，最上层为DecorView，mContentParent为DecorView中id为Window.ID_ANDROID_CONTENT的子View，而我们通过setContentView方法传入的布局文件则是mContentParent的子View。Activity在显示时，会设置ViewRoot中的mView变量：

{% highlight java linenos %}
void makeVisible() {
    if (!mWindowAdded) {
        ViewManager wm = getWindowManager();
        wm.addView(mDecor, getWindow().getAttributes());
        mWindowAdded = true;
    }
    mDecor.setVisibility(View.VISIBLE);
}
{% endhighlight %}

getWindowManager方法返回的是LocalWindowManager类的实例，其内部封装了一个WindowManagerImpl实例。LocalWindowManager的addView方法最终调用了WindowManagerImpl的addView方法：

{% highlight java linenos %}
private void addView(View view, ViewGroup.LayoutParams params, boolean nest)
{

    final WindowManager.LayoutParams wparams
            = (WindowManager.LayoutParams)params;
    
    ViewRoot root;
    View panelParentView = null;
    
    synchronized (this) {

        int index = findViewLocked(view, false);
        if (index >= 0) {
        ......

            return;
        }

        root = new ViewRoot(view.getContext());
        root.mAddNesting = 1;

        view.setLayoutParams(wparams);

        if (mViews == null) {
            index = 1;
            mViews = new View[1];
            mRoots = new ViewRoot[1];
            mParams = new WindowManager.LayoutParams[1];
        } else {
            index = mViews.length + 1;
            Object[] old = mViews;
            mViews = new View[index];
            System.arraycopy(old, 0, mViews, 0, index-1);
            old = mRoots;
            mRoots = new ViewRoot[index];
            System.arraycopy(old, 0, mRoots, 0, index-1);
            old = mParams;
            mParams = new WindowManager.LayoutParams[index];
            System.arraycopy(old, 0, mParams, 0, index-1);
        }
        index--;

        mViews[index] = view;
        mRoots[index] = root;
        mParams[index] = wparams;
    }
    // do this last because it fires off messages to start doing things
    root.setView(view, wparams, panelParentView);
}
{% endhighlight %}

在19行处创建了ViewRoot的实例，然后在48行处调用了ViewRoot的setView方法设置了ViewRoot的mView变量为Activity的mDecorView。

因此对应Activity而言，touch事件首先分发给DecorView的dispatchTouchEvent方法。看下DecorView的dispatchTouchEvent方法：

{% highlight java linenos %}
@Override
public boolean dispatchTouchEvent(MotionEvent ev) {
    final Callback cb = getCallback();
    return cb != null && mFeatureId < 0 ? cb.dispatchTouchEvent(ev) : super
            .dispatchTouchEvent(ev);
}
{% endhighlight %}

这里的Callback就是Activity自身，因此touch事件从DecorView的dispatchTouchEvent方法传递到Activity的dispatchTouchEvent方法：

{% highlight java linenos %}
/**
 * Called to process touch screen events.  You can override this to
 * intercept all touch screen events before they are dispatched to the
 * window.  Be sure to call this implementation for touch screen events
 * that should be handled normally.
 * 
 * @param ev The touch screen event.
 * 
 * @return boolean Return true if this event was consumed.
 */
public boolean dispatchTouchEvent(MotionEvent ev) {
    if (ev.getAction() == MotionEvent.ACTION_DOWN) {
        onUserInteraction();
    }
    if (getWindow().superDispatchTouchEvent(ev)) {
        return true;
    }
    return onTouchEvent(ev);
}
{% endhighlight %}

Activity的dispatchTouchEvent方法接收到touch事件，在15行处分发给PhoneWindow的superDispatchTouchEvent方法；PhoneWindow再将touch事件分发给DecorView的superDispatchTouchEvent方法；而DecorView则调用super.dispatchTouchEvent方法处理touch事件；

{% highlight java linenos %}
public boolean superDispatchTouchEvent(MotionEvent event) {
    return super.dispatchTouchEvent(event);
}
{% endhighlight %}

DecorView是FrameLayout的子类，这样最终touch事件分发给ViewGroup的dispatchTouchEvent方法来处理。

***

#相关文章

[Android Touch事件传递机制(一) -- onInterceptTouchEvent & onTouchEvent][1]

[Android Touch事件传递机制(二) -- OnTouchListener & OnClickListener & OnLongClickListener][2]

***

#参考

[Android事件分发机制完全解析，带你从源码的角度彻底理解(上)](http://blog.csdn.net/guolin_blog/article/details/9097463)

[Android事件分发机制完全解析，带你从源码的角度彻底理解(下)](http://blog.csdn.net/guolin_blog/article/details/9153747)

[1]: {% post_url 2014-08-17-android-onInterceptTouchEvent-onTouchEvent %} "Android Touch事件传递机制(一) -- onInterceptTouchEvent & onTouchEvent"

[2]: {% post_url 2014-08-25-android-OnTouchListener-OnClickListener-OnLongClickListener %} "Android Touch事件传递机制(二) -- OnTouchListener & OnClickListener & OnLongClickListener"