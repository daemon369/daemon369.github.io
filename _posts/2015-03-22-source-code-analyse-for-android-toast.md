---
layout: post
keywords: android, toast
description: source code review for android toast
title: "Android Toast 源码解析"
categories: [Android]
tags: [android, toast]
group: Android
icon: file-alt
date: 2015-03-22 00:00:00
---

Android Toast 的通常用法为：

{% highlight java linenos %}
Toast.makeText(context, "message", Toast.LENGTH_SHORT).show();
{% endhighlight %}

Toast.makeText 源码如下(android-14)：

<!--excerpt-->

{% highlight java linenos %}
/**
 * Make a standard toast that just contains a text view.
 *
 * @param context  The context to use.  Usually your {@link android.app.Application}
 *                 or {@link android.app.Activity} object.
 * @param text     The text to show.  Can be formatted text.
 * @param duration How long to display the message.  Either {@link #LENGTH_SHORT} or
 *                 {@link #LENGTH_LONG}
 *
 */
public static Toast makeText(Context context, CharSequence text, int duration) {
    Toast result = new Toast(context);

    LayoutInflater inflate = (LayoutInflater)
            context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
    View v = inflate.inflate(com.android.internal.R.layout.transient_notification, null);
    TextView tv = (TextView)v.findViewById(com.android.internal.R.id.message);
    tv.setText(text);

    result.mNextView = v;
    result.mDuration = duration;

    return result;
}
{% endhighlight %}

可以看到 makeText 函数新建了个 Toast 对象，并设置变量 mNextView 以及 mDuration 。

mDuration 即 makeText 传入的 Toast 的持续时间， mNextView 的布局文件为 transient_notification ：

{% highlight java linenos %}
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:background="@drawable/toast_frame">

    <TextView
        android:id="@android:id/message"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:textAppearance="@style/TextAppearance.Small"
        android:textColor="@color/bright_foreground_dark"
        android:shadowColor="#BB000000"
        android:shadowRadius="2.75"
        />

</LinearLayout>
{% endhighlight %}

可以看到，Toast 的布局很简单：一个 LinearLayout 之中包含一个 id 为 @android:id/message 的 TextView。

再来看看 Toast 的构造函数：

{% highlight java linenos %}
/**
 * Construct an empty Toast object.  You must call {@link #setView} before you
 * can call {@link #show}.
 *
 * @param context  The context to use.  Usually your {@link android.app.Application}
 *                 or {@link android.app.Activity} object.
 */
public Toast(Context context) {
    mContext = context;
    mTN = new TN();
    mTN.mY = context.getResources().getDimensionPixelSize(
            com.android.internal.R.dimen.toast_y_offset);
}
{% endhighlight %}

可以看到 Toast 的构造函数中创建了一个内部类 TN 的实例：

{% highlight java linenos %}
private static class TN extends ITransientNotification.Stub {
    final Runnable mShow = new Runnable() {
        public void run() {
            handleShow();
        }
    };

    final Runnable mHide = new Runnable() {
        public void run() {
            handleHide();
            // Don't do this in handleHide() because it is also invoked by handleShow()
            mNextView = null;
        }
    };

    private final WindowManager.LayoutParams mParams = new WindowManager.LayoutParams();
    final Handler mHandler = new Handler();

    int mGravity = Gravity.CENTER_HORIZONTAL | Gravity.BOTTOM;
    int mX, mY;
    float mHorizontalMargin;
    float mVerticalMargin;


    View mView;
    View mNextView;

    WindowManagerImpl mWM;

    TN() {
        // XXX This should be changed to use a Dialog, with a Theme.Toast
        // defined that sets up the layout params appropriately.
        final WindowManager.LayoutParams params = mParams;
        params.height = WindowManager.LayoutParams.WRAP_CONTENT;
        params.width = WindowManager.LayoutParams.WRAP_CONTENT;
        params.flags = WindowManager.LayoutParams.FLAG_NOT_FOCUSABLE
                | WindowManager.LayoutParams.FLAG_NOT_TOUCHABLE
                | WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON;
        params.format = PixelFormat.TRANSLUCENT;
        params.windowAnimations = com.android.internal.R.style.Animation_Toast;
        params.type = WindowManager.LayoutParams.TYPE_TOAST;
        params.setTitle("Toast");
    }

    /**
     * schedule handleShow into the right thread
     */
    public void show() {
        if (localLOGV) Log.v(TAG, "SHOW: " + this);
        mHandler.post(mShow);
    }

    /**
     * schedule handleHide into the right thread
     */
    public void hide() {
        if (localLOGV) Log.v(TAG, "HIDE: " + this);
        mHandler.post(mHide);
    }

    public void handleShow() {
        if (localLOGV) Log.v(TAG, "HANDLE SHOW: " + this + " mView=" + mView
                + " mNextView=" + mNextView);
        if (mView != mNextView) {
            // remove the old view if necessary
            handleHide();
            mView = mNextView;
            mWM = WindowManagerImpl.getDefault();
            final int gravity = mGravity;
            mParams.gravity = gravity;
            if ((gravity & Gravity.HORIZONTAL_GRAVITY_MASK) == Gravity.FILL_HORIZONTAL) {
                mParams.horizontalWeight = 1.0f;
            }
            if ((gravity & Gravity.VERTICAL_GRAVITY_MASK) == Gravity.FILL_VERTICAL) {
                mParams.verticalWeight = 1.0f;
            }
            mParams.x = mX;
            mParams.y = mY;
            mParams.verticalMargin = mVerticalMargin;
            mParams.horizontalMargin = mHorizontalMargin;
            if (mView.getParent() != null) {
                if (localLOGV) Log.v(TAG, "REMOVE! " + mView + " in " + this);
                mWM.removeView(mView);
            }
            if (localLOGV) Log.v(TAG, "ADD! " + mView + " in " + this);
            mWM.addView(mView, mParams);
            trySendAccessibilityEvent();
        }
    }

    private void trySendAccessibilityEvent() {
        AccessibilityManager accessibilityManager =
                AccessibilityManager.getInstance(mView.getContext());
        if (!accessibilityManager.isEnabled()) {
            return;
        }
        // treat toasts as notifications since they are used to
        // announce a transient piece of information to the user
        AccessibilityEvent event = AccessibilityEvent.obtain(
                AccessibilityEvent.TYPE_NOTIFICATION_STATE_CHANGED);
        event.setClassName(getClass().getName());
        event.setPackageName(mView.getContext().getPackageName());
        mView.dispatchPopulateAccessibilityEvent(event);
        accessibilityManager.sendAccessibilityEvent(event);
    }

    public void handleHide() {
        if (localLOGV) Log.v(TAG, "HANDLE HIDE: " + this + " mView=" + mView);
        if (mView != null) {
            // note: checking parent() just to make sure the view has
            // been added...  i have seen cases where we get here when
            // the view isn't yet added, so let's try not to crash.
            if (mView.getParent() != null) {
                if (localLOGV) Log.v(TAG, "REMOVE! " + mView + " in " + this);
                mWM.removeView(mView);
            }

            mView = null;
        }
    }
}
{% endhighlight %}

我们来分析下内部类 TN 的源码。

TN 继承于 ITransientNotification.Stub，而 ITransientNotification.aidl 位于 frameworks/base/core/java/android/app/ITransientNotification.aidl：

{% highlight java linenos %}
package android.app;

/** @hide */
oneway interface ITransientNotification {
    void show();
    void hide();
}
{% endhighlight %}

ITransientNotification 中定义了两个方法： show()、hide()，它们在 TN 中的实现为：

{% highlight java linenos %}
public void show() {
    if (localLOGV) Log.v(TAG, "SHOW: " + this);
    mHandler.post(mShow);
}

public void hide() {
    if (localLOGV) Log.v(TAG, "HIDE: " + this);
    mHandler.post(mHide);
}
{% endhighlight %}

这里的 show 和 hide 是通过 Handler 机制来调用具体逻辑的，对应的 Runnable 实现为：

{% highlight java linenos %}
final Runnable mShow = new Runnable() {
    public void run() {
        handleShow();
    }
};

final Runnable mHide = new Runnable() {
    public void run() {
        handleHide();
        // Don't do this in handleHide() because it is also invoked by handleShow()
        mNextView = null;
    }
};
{% endhighlight %}

可以看到，show() 和 hide() 方法分别调用了 handleShow() 和 handleHide() 方法。

首先看下 handleShow()：

{% highlight java linenos %}
public void handleShow() {
    if (localLOGV) Log.v(TAG, "HANDLE SHOW: " + this + " mView=" + mView
            + " mNextView=" + mNextView);
    if (mView != mNextView) {
        // remove the old view if necessary
        handleHide();
        mView = mNextView;
        mWM = WindowManagerImpl.getDefault();
        final int gravity = mGravity;
        mParams.gravity = gravity;
        if ((gravity & Gravity.HORIZONTAL_GRAVITY_MASK) == Gravity.FILL_HORIZONTAL) {
            mParams.horizontalWeight = 1.0f;
        }
        if ((gravity & Gravity.VERTICAL_GRAVITY_MASK) == Gravity.FILL_VERTICAL) {
            mParams.verticalWeight = 1.0f;
        }
        mParams.x = mX;
        mParams.y = mY;
        mParams.verticalMargin = mVerticalMargin;
        mParams.horizontalMargin = mHorizontalMargin;
        if (mView.getParent() != null) {
            if (localLOGV) Log.v(TAG, "REMOVE! " + mView + " in " + this);
            mWM.removeView(mView);
        }
        if (localLOGV) Log.v(TAG, "ADD! " + mView + " in " + this);
        mWM.addView(mView, mParams);
        trySendAccessibilityEvent();
    }
}
{% endhighlight %}

handleShow() 方法就是具体显示 Toast 的代码。

首先移除旧的 View，然后设置 mView 对应的布局参数 mParams。最终调用 mWM 的 addView 方法将 mView 显示到手机屏幕。

mWM 是 WindowManagerImpl 的实例，而 WindowManagerImpl 类对外是不可见的，我们如果想实现自己的 Toast 自己控制 Toast的显示，可以通过如下方式获取 mWM：

{% highlight java linenos %}
mWM = (WindowManager)context.getSystemService(Context.WINDOW_SERVICE);
{% endhighlight %}

handleHide() 方法就是具体移除 Toast 显示的代码，调用的是 mWM 的 removeView() 方法：

{% highlight java linenos %}
public void handleHide() {
    if (localLOGV) Log.v(TAG, "HANDLE HIDE: " + this + " mView=" + mView);
    if (mView != null) {
        // note: checking parent() just to make sure the view has
        // been added...  i have seen cases where we get here when
        // the view isn't yet added, so let's try not to crash.
        if (mView.getParent() != null) {
            if (localLOGV) Log.v(TAG, "REMOVE! " + mView + " in " + this);
            mWM.removeView(mView);
        }

        mView = null;
    }
}
{% endhighlight %}

我们在调用 Toast.makeText() 方法生成 Toast 实例后，需要调用 Toast.show() 方法来调用 Toast 的显示功能：

{% highlight java linenos %}
/**
 * Show the view for the specified duration.
 */
public void show() {
    if (mNextView == null) {
        throw new RuntimeException("setView must have been called");
    }

    INotificationManager service = getService();
    String pkg = mContext.getPackageName();
    TN tn = mTN;
    tn.mNextView = mNextView;

    try {
        service.enqueueToast(pkg, tn, mDuration);
    } catch (RemoteException e) {
        // Empty
    }
}
{% endhighlight %}

可以看到，Toast 的显示并不是立即调用的，而是通过 INotificationManager 实例 sService 的 enqueueToast() 方法，将 mTN 添加到 sService 队列中，由 sService 控制 Toast 的显示与移除。

下面看下 sService：

{% highlight java linenos %}
private static INotificationManager sService;

static private INotificationManager getService() {
    if (sService != null) {
        return sService;
    }
    sService = INotificationManager.Stub.asInterface(ServiceManager.getService("notification"));
    return sService;
}
{% endhighlight %}

这里 INofiticationManager 的具体实现类位于 frameworks/base/services/java/com/android/server/NotificationManagerService.java。

首先看下 NotificationManagerService.enqueueToast()：

{% highlight java linenos %}
public void enqueueToast(String pkg, ITransientNotification callback, int duration)
{
    if (DBG) Slog.i(TAG, "enqueueToast pkg=" + pkg + " callback=" + callback + " duration=" + duration);

    if (pkg == null || callback == null) {
        Slog.e(TAG, "Not doing toast. pkg=" + pkg + " callback=" + callback);
        return ;
    }

    synchronized (mToastQueue) {
        int callingPid = Binder.getCallingPid();
        long callingId = Binder.clearCallingIdentity();
        try {
            ToastRecord record;
            int index = indexOfToastLocked(pkg, callback);
            // If it's already in the queue, we update it in place, we don't
            // move it to the end of the queue.
            if (index >= 0) {
                record = mToastQueue.get(index);
                record.update(duration);
            } else {
                // Limit the number of toasts that any given package except the android
                // package can enqueue.  Prevents DOS attacks and deals with leaks.
                if (!"android".equals(pkg)) {
                    int count = 0;
                    final int N = mToastQueue.size();
                    for (int i=0; i<N; i++) {
                         final ToastRecord r = mToastQueue.get(i);
                         if (r.pkg.equals(pkg)) {
                             count++;
                             if (count >= MAX_PACKAGE_NOTIFICATIONS) {
                                 Slog.e(TAG, "Package has already posted " + count
                                        + " toasts. Not showing more. Package=" + pkg);
                                 return;
                             }
                         }
                    }
                }

                record = new ToastRecord(callingPid, pkg, callback, duration);
                mToastQueue.add(record);
                index = mToastQueue.size() - 1;
                keepProcessAliveLocked(callingPid);
            }
            // If it's at index 0, it's the current toast.  It doesn't matter if it's
            // new or just been updated.  Call back and tell it to show itself.
            // If the callback fails, this will remove it from the list, so don't
            // assume that it's valid after this.
            if (index == 0) {
                showNextToastLocked();
            }
        } finally {
            Binder.restoreCallingIdentity(callingId);
        }
    }
}
{% endhighlight %}

mToastQueue 是一个 ToastRecord 对象队列：

{% highlight java linenos %}
private static final class ToastRecord
{
    final int pid;
    final String pkg;
    final ITransientNotification callback;
    int duration;

    ToastRecord(int pid, String pkg, ITransientNotification callback, int duration)
    {
        this.pid = pid;
        this.pkg = pkg;
        this.callback = callback;
        this.duration = duration;
    }

    void update(int duration) {
        this.duration = duration;
    }

    void dump(PrintWriter pw, String prefix) {
        pw.println(prefix + this);
    }

    @Override
    public final String toString()
    {
        return "ToastRecord{"
            + Integer.toHexString(System.identityHashCode(this))
            + " pkg=" + pkg
            + " callback=" + callback
            + " duration=" + duration;
    }
}
{% endhighlight %}

ToastRecord 中：pid 为生成并调用显示 Toast 的进程的 id 号；pkg 为包名；callback 为 ITransientNotification 类型的回调，即前面提到的 mTN；duration 为 Toast 显示持续时间。

enqueueToast 方法中，首先根据 pkg 和 callback 来判断 Toast 是否已在队列中：

{% highlight java linenos %}
private int indexOfToastLocked(String pkg, ITransientNotification callback)
{
    IBinder cbak = callback.asBinder();
    ArrayList<ToastRecord> list = mToastQueue;
    int len = list.size();
    for (int i=0; i<len; i++) {
        ToastRecord r = list.get(i);
        if (r.pkg.equals(pkg) && r.callback.asBinder() == cbak) {
            return i;
        }
    }
    return -1;
}
{% endhighlight %}

如果在队列中，调用 ToastRecord.update 方法更新 Toast 持续时间。

不在队列中，如果 Toast 不是系统 Toast 且队列超过最大数量，直接返回；否则，新建 ToastRecord 对象添加到 mToastQueue 队列中，并调用 keepProcessAliveLocked() 方法将对应应用进程切换到前台进程，防止进程被回收：

{% highlight java linenos %}
private void keepProcessAliveLocked(int pid)
{
    int toastCount = 0; // toasts from this pid
    ArrayList<ToastRecord> list = mToastQueue;
    int N = list.size();
    for (int i=0; i<N; i++) {
        ToastRecord r = list.get(i);
        if (r.pid == pid) {
            toastCount++;
        }
    }
    try {
        mAm.setProcessForeground(mForegroundToken, pid, toastCount > 0);
    } catch (RemoteException e) {
        // Shouldn't happen.
    }
}
{% endhighlight %}

然后判断 Toast 序号是否为0，为0则调用 showNextToastLocked() 方法开始处理 Toast 队列的显示。首次添加 Toast 时，序号当然为0：

{% highlight java linenos %}
private void showNextToastLocked() {
    ToastRecord record = mToastQueue.get(0);
    while (record != null) {
        if (DBG) Slog.d(TAG, "Show pkg=" + record.pkg + " callback=" + record.callback);
        try {
            record.callback.show();
            scheduleTimeoutLocked(record, false);
            return;
        } catch (RemoteException e) {
            Slog.w(TAG, "Object died trying to show notification " + record.callback
                    + " in package " + record.pkg);
            // remove it from the list and let the process die
            int index = mToastQueue.indexOf(record);
            if (index >= 0) {
                mToastQueue.remove(index);
            }
            keepProcessAliveLocked(record.pid);
            if (mToastQueue.size() > 0) {
                record = mToastQueue.get(0);
            } else {
                record = null;
            }
        }
    }
}
{% endhighlight %}

showNextToastLocked() 方法每次从队列列首取出一个 Toast 记录 record，回调 mTN 的 show() 方法将 Toast 显示到屏幕，然后调用 scheduleTimeoutLocked() 方法：

{% highlight java linenos %}
private void scheduleTimeoutLocked(ToastRecord r, boolean immediate)
{
    Message m = Message.obtain(mHandler, MESSAGE_TIMEOUT, r);
    long delay = immediate ? 0 : (r.duration == Toast.LENGTH_LONG ? LONG_DELAY : SHORT_DELAY);
    mHandler.removeCallbacksAndMessages(r);
    mHandler.sendMessageDelayed(m, delay);
}
{% endhighlight %}

scheduleTimeoutLocked() 方法使用 Handler 机制向 mHandler 的消息队列中添加延迟消息 m，其作用就是在 Toast 的持续时间结束后，从屏幕移除 Toast。

这里可以看到，消息 m 的延时时间根据 duration 的不同，只能为 LONG_DELAY 和 SHORT_DELAY 之一：

{% highlight java linenos %}
private static final int LONG_DELAY = 3500; // 3.5 seconds
private static final int SHORT_DELAY = 2000; // 2 seconds
{% endhighlight %}

因此，Toast 的持续时间是无法随意定义的，只能长短二选一。

我们再来看看延时时间到达后，Toast 队列的后续处理：

{% highlight java linenos %}
private final class WorkerHandler extends Handler
{
    @Override
    public void handleMessage(Message msg)
    {
        switch (msg.what)
        {
            case MESSAGE_TIMEOUT:
                handleTimeout((ToastRecord)msg.obj);
                break;
        }
    }
}

private void handleTimeout(ToastRecord record)
{
    if (DBG) Slog.d(TAG, "Timeout pkg=" + record.pkg + " callback=" + record.callback);
    synchronized (mToastQueue) {
        int index = indexOfToastLocked(record.pkg, record.callback);
        if (index >= 0) {
            cancelToastLocked(index);
        }
    }
}
{% endhighlight %}

可以看到最终调用了 cancelToastLocked() 方法：

{% highlight java linenos %}
private void cancelToastLocked(int index) {
    ToastRecord record = mToastQueue.get(index);
    try {
        record.callback.hide();
    } catch (RemoteException e) {
        Slog.w(TAG, "Object died trying to hide notification " + record.callback
                + " in package " + record.pkg);
        // don't worry about this, we're about to remove it from
        // the list anyway
    }
    mToastQueue.remove(index);
    keepProcessAliveLocked(record.pid);
    if (mToastQueue.size() > 0) {
        // Show the next one. If the callback fails, this will remove
        // it from the list, so don't assume that the list hasn't changed
        // after this point.
        showNextToastLocked();
    }
}
{% endhighlight %}

首先回调 mTN 的 hide() 方法将 Toast 从屏幕移除；然后从 mToastQueue 队列中移除 Toast；此时，如果 mToastQueue 队列非空，调用 showNextToastLocked() 方法处理下一个 Toast。

这样，Toast 显示、隐藏、队列处理的逻辑就都明了了。

# 参考：

[Toast](https://code.google.com/p/android-source-browsing/source/browse/core/java/android/widget/Toast.java?repo=platform--frameworks--base&name=android-4.0.2_r1 "Toast")

[NotificationManagerService](https://code.google.com/p/android-source-browsing/source/browse/services/java/com/android/server/NotificationManagerService.java?repo=platform--frameworks--base&name=android-4.0.2_r1 "NotificationManagerService")

[Android Toast源码分析](http://blog.csdn.net/wzy_1988/article/details/43341761 "Android Toast源码分析")
