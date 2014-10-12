---
layout: post
keywords: andrid, onInterceptTouchEvent, onTouchEvent, dispatchTouchEvent
description: onTouchEvent
title: "Android Touch事件传递机制(四) -- Touch事件处理(onTouchEvent)"
categories: [Android]
tags: [android, OnTouchListener, onTouch, onTouchEvent]
group: Android
icon: file-alt
---
{% include JB/setup %}

***

(本文基于android-2.3.3_r1代码研究)

前面研究了Android触屏事件的分发机制；本文继续从源码角度研究触屏事件的处理机制。

***

#一. View.onTouchEvent

Android的触屏事件，最终分发给View的onTouchEvent方法来处理，我们来看看它是怎么处理的：

<!--excerpt-->

{% highlight java linenos %}
/**
 * Implement this method to handle touch screen motion events.
 *
 * @param event The motion event.
 * @return True if the event was handled, false otherwise.
 */
public boolean onTouchEvent(MotionEvent event) {
    final int viewFlags = mViewFlags;

    if ((viewFlags & ENABLED_MASK) == DISABLED) {
        // A disabled view that is clickable still consumes the touch
        // events, it just doesn't respond to them.
        return (((viewFlags & CLICKABLE) == CLICKABLE ||
                (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE));
    }

    if (mTouchDelegate != null) {
        if (mTouchDelegate.onTouchEvent(event)) {
            return true;
        }
    }

    if (((viewFlags & CLICKABLE) == CLICKABLE ||
            (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE)) {
        switch (event.getAction()) {
            case MotionEvent.ACTION_UP:
                boolean prepressed = (mPrivateFlags & PREPRESSED) != 0;
                if ((mPrivateFlags & PRESSED) != 0 || prepressed) {
                    // take focus if we don't have it already and we should in
                    // touch mode.
                    boolean focusTaken = false;
                    if (isFocusable() && isFocusableInTouchMode() && !isFocused()) {
                        focusTaken = requestFocus();
                    }

                    if (!mHasPerformedLongPress) {
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
                        mPrivateFlags |= PRESSED;
                        refreshDrawableState();
                        postDelayed(mUnsetPressedState,
                                ViewConfiguration.getPressedStateDuration());
                    } else if (!post(mUnsetPressedState)) {
                        // If the post failed, unpress right now
                        mUnsetPressedState.run();
                    }
                    removeTapCallback();
                }
                break;

            case MotionEvent.ACTION_DOWN:
                if (mPendingCheckForTap == null) {
                    mPendingCheckForTap = new CheckForTap();
                }
                mPrivateFlags |= PREPRESSED;
                mHasPerformedLongPress = false;
                postDelayed(mPendingCheckForTap, ViewConfiguration.getTapTimeout());
                break;

            case MotionEvent.ACTION_CANCEL:
                mPrivateFlags &= ~PRESSED;
                refreshDrawableState();
                removeTapCallback();
                break;

            case MotionEvent.ACTION_MOVE:
                final int x = (int) event.getX();
                final int y = (int) event.getY();

                // Be lenient about moving outside of buttons
                int slop = mTouchSlop;
                if ((x < 0 - slop) || (x >= getWidth() + slop) ||
                        (y < 0 - slop) || (y >= getHeight() + slop)) {
                    // Outside button
                    removeTapCallback();
                    if ((mPrivateFlags & PRESSED) != 0) {
                        // Remove any future long press/tap checks
                        removeLongPressCallback();

                        // Need to switch from pressed to not pressed
                        mPrivateFlags &= ~PRESSED;
                        refreshDrawableState();
                    }
                }
                break;
        }
        return true;
    }

    return false;
}
{% endhighlight %}

由第13行及第23行代码可知，当View的CLICKABLE状态位或LONG_CLICKABLE状态位为1时，即View有可点击或可长按时，onTouchEvent方法返回true；由前面的文章可知，onTouchEvent方法返回true就代表View消费了touch事件；而View的setOnClickListener方法会设置CLICKABLE状态位，setOnLongClickListener会设置LONG_CLICKABLE状态位。因此，当对View设置了OnClickListener或OnLongClickListener时，这个View就会消费父ViewGroup分发的touch事件。

由第10行处代码可知，当View是DISABLED状态时，仍然会消费分发过来的touch事件，只是不做任何处理。

###1. down事件

首先看下对down事件的处理：

{% highlight java linenos %}
    case MotionEvent.ACTION_DOWN:
        if (mPendingCheckForTap == null) {
            mPendingCheckForTap = new CheckForTap();
        }
        mPrivateFlags |= PREPRESSED;
        mHasPerformedLongPress = false;
        postDelayed(mPendingCheckForTap, ViewConfiguration.getTapTimeout());
        break;
{% endhighlight %}
                    
首先创建了一个CheckForTap对象mPendingCheckForTap，CheckForTap是个Runnable接口实例；然后设置了PREPRESSED状态位；最后调用View.postDelayed方法将tap延时消息发送到主线程消息队列，延迟时间为ViewConfiguration.getTapTimeout() (android2.3.3_r1中是115ms)：

{% highlight java linenos %}
private final class CheckForTap implements Runnable {
    public void run() {
        mPrivateFlags &= ~PREPRESSED;
        mPrivateFlags |= PRESSED;
        refreshDrawableState();
        if ((mViewFlags & LONG_CLICKABLE) == LONG_CLICKABLE) {
            postCheckForLongClick(ViewConfiguration.getTapTimeout());
        }
    }
}
{% endhighlight %}

当用户按下触屏并保持一定时间后，就会执行CheckForTap的run方法：首先取消了PREPRESSED状态位，设置了PRESSED状态位；然后刷新View的drawable状态；如果View可长按，则调用postCheckForLongClick方法：

{% highlight java linenos %}
private void postCheckForLongClick(int delayOffset) {
    mHasPerformedLongPress = false;

    if (mPendingCheckForLongPress == null) {
        mPendingCheckForLongPress = new CheckForLongPress();
    }
    mPendingCheckForLongPress.rememberWindowAttachCount();
    postDelayed(mPendingCheckForLongPress,
            ViewConfiguration.getLongPressTimeout() - delayOffset);
}
{% endhighlight %}

postCheckForLongClick方法中创建了CheckForLongPress对象mPendingCheckForLongPress，然后发送long click延时消息到消息队列，这次的延迟时间为ViewConfiguration.getLongPressTimeout() - ViewConfiguration.getTapTimeout()， 即长按时间减去tap时间(android2.3.3_r1中为 500ms-115ms)。CheckForLongPress同CheckForTap类似，也是一个Runnable接口实例，只不过CheckForTap是用来处理tap事件，CheckForLongPress用来处理长按事件。

{% highlight java linenos %}
class CheckForLongPress implements Runnable {

    private int mOriginalWindowAttachCount;

    public void run() {
        if (isPressed() && (mParent != null)
                && mOriginalWindowAttachCount == mWindowAttachCount) {
            if (performLongClick()) {
                mHasPerformedLongPress = true;
            }
        }
    }

    public void rememberWindowAttachCount() {
        mOriginalWindowAttachCount = mWindowAttachCount;
    }
}
{% endhighlight %}

当用户按下View的时间超过长按时间阈值时，就会调用CheckForLongPress.run()方法；在run方法中调用View.performLongClick()方法，处理用户长按事件。

###2. move事件

下面再看下onTouchEvent方法对move事件的处理：

{% highlight java linenos %}
    case MotionEvent.ACTION_MOVE:
        final int x = (int) event.getX();
        final int y = (int) event.getY();

        // Be lenient about moving outside of buttons
        int slop = mTouchSlop;
        if ((x < 0 - slop) || (x >= getWidth() + slop) ||
                (y < 0 - slop) || (y >= getHeight() + slop)) {
            // Outside button
            removeTapCallback();
            if ((mPrivateFlags & PRESSED) != 0) {
                // Remove any future long press/tap checks
                removeLongPressCallback();

                // Need to switch from pressed to not pressed
                mPrivateFlags &= ~PRESSED;
                refreshDrawableState();
            }
        }
        break;
{% endhighlight %}

move事件的处理很简单：View接收到move事件时，如果触点移出View时，调用removeTapCallback和removeLongPressCallback，将发送到主线程消息队列的tap、long click消息移除，不再处理点击或长按事件，同时取消PRESSED状态。

###3. up事件

再来看下onTouchEvent方法对up事件的处理：

{% highlight java linenos %}
    case MotionEvent.ACTION_UP:
        boolean prepressed = (mPrivateFlags & PREPRESSED) != 0;
        if ((mPrivateFlags & PRESSED) != 0 || prepressed) {
            // take focus if we don't have it already and we should in
            // touch mode.
            boolean focusTaken = false;
            if (isFocusable() && isFocusableInTouchMode() && !isFocused()) {
                focusTaken = requestFocus();
            }

            if (!mHasPerformedLongPress) {
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
                mPrivateFlags |= PRESSED;
                refreshDrawableState();
                postDelayed(mUnsetPressedState,
                        ViewConfiguration.getPressedStateDuration());
            } else if (!post(mUnsetPressedState)) {
                // If the post failed, unpress right now
                mUnsetPressedState.run();
            }
            removeTapCallback();
        }
        break;
{% endhighlight %}

第8行处，首先View请求焦点；20到25行处理点击事件，最终会调用mOnClickListener的onClick方法；29行到41行处取消View的PRESSED状态。

其中，第11行处会判断mHasPerformedLongPress的值，如果mHasPerformedLongPress为true，则不会处理用户点击事件；mHasPerformedLongPress为false才会处理。那么mHasPerformedLongPress变量是在哪里设置的呢？

mHasPerformedLongPress初始为false，在View接收到down事件时以及发送long click延时消息时都会设置为false；只有在处理长按事件时，调用View.performLongClick()方法返回为true时，才会设置为true。我们看下View.performLongClick()方法：

{% highlight java linenos %}
/**
 * Call this view's OnLongClickListener, if it is defined. Invokes the context menu if the
 * OnLongClickListener did not consume the event.
 *
 * @return True if one of the above receivers consumed the event, false otherwise.
 */
public boolean performLongClick() {
    sendAccessibilityEvent(AccessibilityEvent.TYPE_VIEW_LONG_CLICKED);

    boolean handled = false;
    if (mOnLongClickListener != null) {
        handled = mOnLongClickListener.onLongClick(View.this);
    }
    if (!handled) {
        handled = showContextMenu();
    }
    if (handled) {
        performHapticFeedback(HapticFeedbackConstants.LONG_PRESS);
    }
    return handled;
}
{% endhighlight %}

可以看到，只有在回调mOnLongClickListener.onLongClick方法返回true时，performLongClick才会返回true。这就意味着当给View设置了OnLongClickListener且OnLongClickListener.onLongClick方法返回true时，长按View后松开，点击事件不再触发；当OnLongClickListener.onLongClick方法返回false时，长按View时触发长按事件，然后松开触屏后，点击事件依旧会被触发。这在[前面的文章][2]中已经用demo验证过的。

***

#相关文章

[Android Touch事件传递机制(一) -- onInterceptTouchEvent & onTouchEvent][1]

[Android Touch事件传递机制(二) -- OnTouchListener & OnClickListener & OnLongClickListener][2]

[Android Touch事件传递机制(三) -- Touch事件分发(dispatchTouchEvent)][3]

***

#参考

[Android事件分发机制完全解析，带你从源码的角度彻底理解(上)](http://blog.csdn.net/guolin_blog/article/details/9097463)

[Android事件分发机制完全解析，带你从源码的角度彻底理解(下)](http://blog.csdn.net/guolin_blog/article/details/9153747)

[1]: {% post_url 2014-08-17-android-onInterceptTouchEvent-onTouchEvent %} "Android Touch事件传递机制(一) -- onInterceptTouchEvent & onTouchEvent"

[2]: {% post_url 2014-08-25-android-OnTouchListener-OnClickListener-OnLongClickListener %} "Android Touch事件传递机制(二) -- OnTouchListener & OnClickListener & OnLongClickListener"

[3]: {% post_url 2014-09-11-android-dispatchTouchEvent %} "Android Touch事件传递机制(三) -- Touch事件分发(dispatchTouchEvent)"