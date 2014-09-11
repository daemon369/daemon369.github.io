---
layout: post
keywords: andrid, onInterceptTouchEvent, onTouchEvent
description: onInterceptTouchEvent & onTouchEvent
title: "Android Touch事件传递机制(一) -- onInterceptTouchEvent & onTouchEvent"
categories: [Android]
tags: [android, onInterceptTouchEvent, onTouchEvent]
group: Android
icon: file-alt
---
{% include JB/setup %}

***

(本文基于android-2.3.3_r1代码研究)

#onTouchEvent

onTouchEvent是View类提供的方法，其说明如下：

{% highlight java %}
 /**
 * Implement this method to handle touch screen motion events.
 * <p>
 * If this method is used to detect click actions, it is recommended that
 * the actions be performed by implementing and calling
 * {@link #performClick()}. This will ensure consistent system behavior,
 * including:
 * <ul>
 * <li>obeying click sound preferences
 * <li>dispatching OnClickListener calls
 * <li>handling {@link AccessibilityNodeInfo#ACTION_CLICK ACTION_CLICK} when
 * accessibility features are enabled
 * </ul>
 *
 * @param event The motion event.
 * @return True if the event was handled, false otherwise.
 */
public boolean onTouchEvent(MotionEvent event);
{% endhighlight %}

<!--excerpt-->

实现onTouchEvent方法可以处理View的触屏事件，返回true表明事件已被处理，不需要继续派发，否则返回false。

###Demo

如下自定义View布局文件

{% highlight java %}
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <com.daemon.demo.touchdemo.widget.MyView
        android:layout_width="100dp"
        android:layout_height="100dp"
        android:background="#ff0000" />

</RelativeLayout>
{% endhighlight %}

在MyView上按下触屏并滑动

###1. MyView中onTouchEvent返回false

    E/MyView﹕ onTouchEvent:false action:ACTION_DOWN
    
可以看到onTouchEvent返回false时，只会收到第一次的down事件，后续的move、up事件都不再派发给MyView

###2. MyView中onTouchEvent返回true

    E/MyView﹕ onTouchEvent:true action:ACTION_DOWN
    E/MyView﹕ onTouchEvent:true action:ACTION_MOVE
    E/MyView﹕ onTouchEvent:true action:ACTION_MOVE

    ......

    E/MyView﹕ onTouchEvent:true action:ACTION_MOVE
    E/MyView﹕ onTouchEvent:true action:ACTION_UP

可以看到onTouchEvent返回true时，所有的down、move、up事件都会派发给MyView处理

###总结

1. onTouchEvent返回true，父ViewGroup派发过来的touch事件已被该View消费，不会再向上传递给父ViewGroup；后续的touch事件都将继续传递给View

2. onTouchEvent返回false，表明View并不消费父ViewGroup传递来的down事件，而是向上传递给父ViewGroup来处理；后续的move、up等事件将不再传递给View，直接由父ViewGroup处理掉

***

#onInterceptTouchEvent

onInterceptTouchEvent()方法是ViewGoup中的方法，其注释如下：

{% highlight java %}
 /**
 * Implement this method to intercept all touch screen motion events.  This
 * allows you to watch events as they are dispatched to your children, and
 * take ownership of the current gesture at any point.
 *
 * <p>Using this function takes some care, as it has a fairly complicated
 * interaction with {@link View#onTouchEvent(MotionEvent)
 * View.onTouchEvent(MotionEvent)}, and using it requires implementing
 * that method as well as this one in the correct way.  Events will be
 * received in the following order:
 *
 * <ol>
 * <li> You will receive the down event here.
 * <li> The down event will be handled either by a child of this view
 * group, or given to your own onTouchEvent() method to handle; this means
 * you should implement onTouchEvent() to return true, so you will
 * continue to see the rest of the gesture (instead of looking for
 * a parent view to handle it).  Also, by returning true from
 * onTouchEvent(), you will not receive any following
 * events in onInterceptTouchEvent() and all touch processing must
 * happen in onTouchEvent() like normal.
 * <li> For as long as you return false from this function, each following
 * event (up to and including the final up) will be delivered first here
 * and then to the target's onTouchEvent().
 * <li> If you return true from here, you will not receive any
 * following events: the target view will receive the same event but
 * with the action {@link MotionEvent#ACTION_CANCEL}, and all further
 * events will be delivered to your onTouchEvent() method and no longer
 * appear here.
 * </ol>
 *
 * @param ev The motion event being dispatched down the hierarchy.
 * @return Return true to steal motion events from the children and have
 * them dispatched to this ViewGroup through onTouchEvent().
 * The current target will receive an ACTION_CANCEL event, and no further
 * messages will be delivered here.
 */
public boolean onInterceptTouchEvent(MotionEvent ev);
{% endhighlight %}

实现onInterceptTouchEvent方法可以用来拦截父ViewGroup传递下来的所有触屏事件，可以将所有触屏事件交由此ViewGroup自身的onTouchEvent来处理，也可以继续传递给其子View来处理。

onInterceptTouchEvent方法对触屏事件的拦截处理需要和onTouchEvent方法配合使用。

1. down事件首先传递到onInterceptTouchEvent方法中

2. onInterceptTouchEvent返回false表示将down事件交由子View来处理；若某一层子View的onTouchEvent返回了true，后续的move、up等事件都将先传递到ViewGroup的onInterceptTouchEvent的方法，并继续层层传递下去，交由子View处理；若子View的onTouchEvent都返回了false，则down事件将交由该ViewGroup的onTouchEvent来处理；如果ViewGroup的onTouchEvent返回false，down传递给父ViewGroup，后续事件不再传递给该ViewGroup；如果ViewGroup的onTouchEvent返回true，后续事件不再经过该ViewGroup的onInterceptTouchEvent方法，直接传递给onTouchEvent方法处理

3. onInterceptTouchEvent返回ture，down事件将转交该ViewGroup的onTouchEvent来处理；若onTouchEvent返回true，后续事件将不再经过该ViewGroup的onInterceptTouchEvent方法，直接交由该ViewGroup的onTouchEvent方法处理；若onTouchEvent方法返回false，后续事件都将交由父ViewGroup处理，不再经过该ViewGroup的onInterceptTouchEvent方法和onTouchEvent方法

###一. 没有子View的ViewGroup

{% highlight java %}
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <com.daemon.demo.touchdemo.widget.MyLayout
        android:layout_width="200dp"
        android:layout_height="200dp"
        android:background="#00ff00" />

</RelativeLayout>
{% endhighlight %}

#####1. MyLayout的onInterceptTouchEvent返回false，onTouchEvent返回false

    E/MyLayout﹕ onInterceptTouchEvent:false action:ACTION_DOWN
    E/MyLayout﹕ onTouchEvent:false action:ACTION_DOWN

MyLayout的down事件首先传递到onInterceptTouchEvent方法；

由于没有子View，down传递给MyLayout的onTouchEvent方法来处理；

onTouchEvent返回了false，表示MyLayout不处理touch事件，down事件传递给父ViewGroup的onTouchEvent方法处理，后续的move、up等事件都不再传递给MyLayout，直接交由父ViewGroup处理。

#####2. MyLayout的onInterceptTouchEvent返回false，onTouchEvent返回true

    E/MyLayout﹕ onInterceptTouchEvent:false action:ACTION_DOWN
    E/MyLayout﹕ onTouchEvent:true action:ACTION_DOWN
    E/MyLayout﹕ onTouchEvent:true action:ACTION_MOVE
    E/MyLayout﹕ onTouchEvent:true action:ACTION_MOVE

    ......

    E/MyLayout﹕ onTouchEvent:true action:ACTION_MOVE
    E/MyLayout﹕ onTouchEvent:true action:ACTION_UP

MyLayout的down事件首先传递到onInterceptTouchEvent方法；

由于没有子View，down传递给MyLayout的onTouchEvent方法来处理；

onTouchEvent返回true，down事件不再向上传递，后续move、up等事件不再传递给onInterceptTouchEvent方法，直接交由onTouchEvent处理

#####3. MyLayout的onInterceptTouchEvent返回true，onTouchEvent返回false

    E/MyLayout﹕ onInterceptTouchEvent:true action:ACTION_DOWN
    E/MyLayout﹕ onTouchEvent:false action:ACTION_DOWN

MyLayout的down事件首先传递到onInterceptTouchEvent方法；

onInterceptTouchEvent方法返回true，down事件直接传递给MyLayout的onTouchEvent方法来处理；

onTouchEvent返回了false，表示MyLayout不处理touch事件，down事件传递给父ViewGroup的onTouchEvent方法处理，后续的move、up等事件都不再传递给MyLayout，直接交由父ViewGroup处理。

#####4. MyLayout的onInterceptTouchEvent返回true，onTouchEvent返回true

    E/MyLayout﹕ onInterceptTouchEvent:true action:ACTION_DOWN
    E/MyLayout﹕ onTouchEvent:true action:ACTION_DOWN
    E/MyLayout﹕ onTouchEvent:true action:ACTION_MOVE
    E/MyLayout﹕ onTouchEvent:true action:ACTION_MOVE

    ......

    E/MyLayout﹕ onTouchEvent:true action:ACTION_MOVE
    E/MyLayout﹕ onTouchEvent:true action:ACTION_UP

MyLayout的down事件首先传递到onInterceptTouchEvent方法；

onInterceptTouchEvent方法返回true，down事件直接传递给MyLayout的onTouchEvent方法来处理；

onTouchEvent返回true，down事件不再向上传递，后续move、up等事件不再传递给onInterceptTouchEvent方法，直接交由onTouchEvent处理

###二. ViewGroup嵌套View

布局如下：

{% highlight java %}
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <com.daemon.demo.touchdemo.widget.MyLayout
        android:layout_width="200dp"
        android:layout_height="200dp"
        android:background="#00ff00">

        <com.daemon.demo.touchdemo.widget.MyView
            android:layout_width="100dp"
            android:layout_height="100dp"
            android:background="#ff0000" />

    </com.daemon.demo.touchdemo.widget.MyLayout>

</RelativeLayout>
{% endhighlight %}

#####1.

MyLayout的onInterceptTouchEvent返回false，onTouchEvent返回false

MyView的onTouchEvent返回false

    E/MyLayout﹕ onInterceptTouchEvent:false action:ACTION_DOWN
    E/MyView﹕ onTouchEvent:false action:ACTION_DOWN
    E/MyLayout﹕ onTouchEvent:false action:ACTION_DOWN

MyLayout的down事件首先传递到onInterceptTouchEvent方法；

onInterceptTouchEvent方法返回false，down传递给MyView的onTouchEvent方法来处理；

MyView的onTouchEvent返回了false，表示MyView不处理down事件，down事件传递给MyLayout的onTouchEvent方法处理；

MyLayout的onTouchEvent返回false，down事件继续传递给其父ViewGroup的onTouchEvent方法处理，后续的move、up等事件都不再传递给MyLayout，直接交由父ViewGroup处理。

#####2.

MyLayout的onInterceptTouchEvent返回false，onTouchEvent返回true/false

MyView的onTouchEvent返回true

    E/MyLayout﹕ onInterceptTouchEvent:false action:ACTION_DOWN
    E/MyView﹕ onTouchEvent:true action:ACTION_DOWN
    E/MyLayout﹕ onInterceptTouchEvent:false action:ACTION_MOVE
    E/MyView﹕ onTouchEvent:true action:ACTION_MOVE
    
    ......
    
    E/MyLayout﹕ onInterceptTouchEvent:false action:ACTION_MOVE
    E/MyView﹕ onTouchEvent:true action:ACTION_MOVE
    E/MyLayout﹕ onInterceptTouchEvent:false action:ACTION_UP
    E/MyView﹕ onTouchEvent:true action:ACTION_UP

MyLayout的down事件首先传递到onInterceptTouchEvent方法；

onInterceptTouchEvent方法返回false，down传递给MyView的onTouchEvent方法来处理；

MyView的onTouchEvent返回了true，表示MyView消费了down事件，不再向上传递，MyLayout的onTouchEvent的返回值此时不起作用

后续的move、up等事件继续由MyLayout的onInterceptTouchEvent方法传递到MyView的onTouchEvent方法进行处理

#####3.

MyLayout的onInterceptTouchEvent返回true，onTouchEvent返回false

MyView的onTouchEvent返回true/false

    E/MyLayout﹕ onInterceptTouchEvent:true action:ACTION_DOWN
    E/MyLayout﹕ onTouchEvent:false action:ACTION_DOWN

MyLayout的down事件首先传递到onInterceptTouchEvent方法；

onInterceptTouchEvent方法返回true，down直接传递给MyLayout的onTouchEvent方法来处理；MyView的onTouchEvent方法的返回值不起作用

MyLayout的onTouchEvent返回false，down事件继续传递给其父ViewGroup的onTouchEvent方法处理，后续的move、up等事件都不再传递给MyLayout，直接交由父ViewGroup处理。

#####4.

MyLayout的onInterceptTouchEvent返回true，onTouchEvent返回true

MyView的onTouchEvent返回true/false

    E/MyLayout﹕ onInterceptTouchEvent:true action:ACTION_DOWN
    E/MyLayout﹕ onTouchEvent:true action:ACTION_DOWN
    E/MyLayout﹕ onTouchEvent:true action:ACTION_MOVE
    E/MyLayout﹕ onTouchEvent:true action:ACTION_MOVE

    ......

    E/MyLayout﹕ onTouchEvent:true action:ACTION_MOVE
    E/MyLayout﹕ onTouchEvent:true action:ACTION_UP

MyLayout的down事件首先传递到onInterceptTouchEvent方法；

onInterceptTouchEvent方法返回true，down直接传递给MyLayout的onTouchEvent方法来处理；MyView的onTouchEvent方法的返回值不起作用

MyLayout的onTouchEvent返回true，down事件由MyLayout消费，不再向上传递；

后续的move、up等事件将不再传递给MyLayout的onInterceptTouchEvent方法，而是直接传递给MyLayout的onTouchEvent方法来处理

###二. ViewGroup嵌套ViewGroup

布局如下：

{% highlight java %}
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <com.daemon.demo.touchdemo.widget.MyLayout
        android:layout_width="200dp"
        android:layout_height="200dp"
        android:background="#00ff00">

        <com.daemon.demo.touchdemo.widget.MyLayout2
            android:layout_width="100dp"
            android:layout_height="100dp"
            android:background="#0000ff" />

    </com.daemon.demo.touchdemo.widget.MyLayout>

</RelativeLayout>
{% endhighlight %}

#####1.

MyLayout的onInterceptTouchEvent返回false，onTouchEvent返回false

MyLayout2的onInterceptTouchEvent返回false，onTouchEvent返回false

    E/MyLayout﹕ onInterceptTouchEvent:false action:ACTION_DOWN
    E/MyLayout2﹕ onInterceptTouchEvent:false action:ACTION_DOWN
    E/MyLayout2﹕ onTouchEvent:false action:ACTION_DOWN
    E/MyLayout﹕ onTouchEvent:false action:ACTION_DOWN

MyLayout的down事件首先传递到onInterceptTouchEvent方法；

onInterceptTouchEvent方法返回false，down传递给MyLayout2的onInterceptTouchEvent方法处理；

由于MyLayout2没有子View，down事件传递给MyLayout2的onTouchEvent方法来处理；

MyLayout2的onTouchEvent返回了false，表示MyLayout2不处理down事件，down事件传递给MyLayout的onTouchEvent方法处理；

MyLayout的onTouchEvent返回false，down事件继续传递给其父ViewGroup的onTouchEvent方法处理，后续的move、up等事件都不再传递给MyLayout，直接交由父ViewGroup处理。

#####2.

MyLayout的onInterceptTouchEvent返回false，onTouchEvent返回true/false

MyLayout2的onInterceptTouchEvent返回false，的onTouchEvent返回true

    E/MyLayout﹕ onInterceptTouchEvent:false action:ACTION_DOWN
    E/MyLayout2﹕ onInterceptTouchEvent:false action:ACTION_DOWN
    E/MyLayout2﹕ onTouchEvent:true action:ACTION_DOWN
    E/MyLayout﹕ onInterceptTouchEvent:false action:ACTION_MOVE
    E/MyLayout2﹕ onTouchEvent:true action:ACTION_MOVE
    E/MyLayout﹕ onInterceptTouchEvent:false action:ACTION_MOVE
    E/MyLayout2﹕ onTouchEvent:true action:ACTION_MOVE
    
    ...
    
    E/MyLayout﹕ onInterceptTouchEvent:false action:ACTION_MOVE
    E/MyLayout2﹕ onTouchEvent:true action:ACTION_MOVE
    E/MyLayout﹕ onInterceptTouchEvent:false action:ACTION_UP
    E/MyLayout2﹕ onTouchEvent:true action:ACTION_UP

MyLayout的down事件首先传递到onInterceptTouchEvent方法；

onInterceptTouchEvent方法返回false，down传递给MyLayout2的onInterceptTouchEvent方法处理；

由于MyLayout2没有子View，down事件传递给MyLayout2的onTouchEvent方法来处理；

MyLayout2的onTouchEvent返回了true，表示MyLayout2消费了down事件，不再向上传递，MyLayout的onTouchEvent的返回值此时不起作用

后续的move、up等事件继续由MyLayout的onInterceptTouchEvent方法传递到MyLayout2，不再经过MyLayout2的onInterceptTouchEvent方法，直接传递给MyLayout2的onTouchEvent方法进行处理

#####3.

MyLayout的onInterceptTouchEvent返回false，onTouchEvent返回false

MyLayout2的onInterceptTouchEvent返回true，onTouchEvent返回false

    E/MyLayout﹕ onInterceptTouchEvent:false action:ACTION_DOWN
    E/MyLayout2﹕ onInterceptTouchEvent:true action:ACTION_DOWN
    E/MyLayout2﹕ onTouchEvent:false action:ACTION_DOWN
    E/MyLayout﹕ onTouchEvent:false action:ACTION_DOWN

MyLayout的down事件首先传递到onInterceptTouchEvent方法；

onInterceptTouchEvent方法返回false，down传递给MyLayout2的onInterceptTouchEvent方法处理；

MyLayout2的onInterceptTouchEvent方法返回true，down事件直接传递给MyLayout2的onTouchEvent方法来处理；

MyLayout2的onTouchEvent返回了false，表示MyLayout2不处理down事件，down事件传递给MyLayout的onTouchEvent方法处理；

MyLayout的onTouchEvent返回false，down事件继续传递给其父ViewGroup的onTouchEvent方法处理，后续的move、up等事件都不再传递给MyLayout，直接交由父ViewGroup处理。

#####4.

MyLayout的onInterceptTouchEvent返回false，onTouchEvent返回true/false

MyLayout2的onInterceptTouchEvent返回true，onTouchEvent返回true

    E/MyLayout﹕ onInterceptTouchEvent:false action:ACTION_DOWN
    E/MyLayout2﹕ onInterceptTouchEvent:true action:ACTION_DOWN
    E/MyLayout2﹕ onTouchEvent:true action:ACTION_DOWN
    E/MyLayout﹕ onInterceptTouchEvent:false action:ACTION_MOVE
    E/MyLayout2﹕ onTouchEvent:true action:ACTION_MOVE
    E/MyLayout﹕ onInterceptTouchEvent:false action:ACTION_MOVE
    E/MyLayout2﹕ onTouchEvent:true action:ACTION_MOVE
    
    ......
    
    E/MyLayout﹕ onInterceptTouchEvent:false action:ACTION_MOVE
    E/MyLayout2﹕ onTouchEvent:true action:ACTION_MOVE
    E/MyLayout﹕ onInterceptTouchEvent:false action:ACTION_UP
    E/MyLayout2﹕ onTouchEvent:true action:ACTION_UP

MyLayout的down事件首先传递到onInterceptTouchEvent方法；

onInterceptTouchEvent方法返回false，down传递给MyLayout2的onInterceptTouchEvent方法处理；

MyLayout2的onInterceptTouchEvent方法返回true，down事件直接传递给MyLayout2的onTouchEvent方法来处理；

MyLayout2的onTouchEvent返回了true，表示MyLayout2消费了down事件，不再向上传递，MyLayout的onTouchEvent的返回值此时不起作用

后续的move、up等事件继续由MyLayout的onInterceptTouchEvent方法传递到MyLayout2，不再经过MyLayout2的onInterceptTouchEvent方法，直接传递给MyLayout2的onTouchEvent方法进行处理

#####5.

MyLayout的onInterceptTouchEvent返回true，onTouchEvent返回false

MyLayout2的onInterceptTouchEvent返回true/false，onTouchEvent返回true/false

    E/MyLayout﹕ onInterceptTouchEvent:true action:ACTION_DOWN
    E/MyLayout﹕ onTouchEvent:false action:ACTION_DOWN

MyLayout的down事件首先传递到onInterceptTouchEvent方法；

onInterceptTouchEvent方法返回true，down直接传递给MyLayout的onTouchEvent方法处理，不传递给MyLayout2；MyLayout2的onInterceptTouchEvent方法和onTouchEvent方法返回值不影响事件传递

MyLayout的onTouchEvent返回false，down事件继续传递给其父ViewGroup的onTouchEvent方法处理，后续的move、up等事件都不再传递给MyLayout，直接交由父ViewGroup处理。

#####6.

MyLayout的onInterceptTouchEvent返回true，onTouchEvent返回true

MyLayout2的onInterceptTouchEvent返回true/false，onTouchEvent返回true/false

    E/MyLayout﹕ onInterceptTouchEvent:true action:ACTION_DOWN
    E/MyLayout﹕ onTouchEvent:true action:ACTION_DOWN
    E/MyLayout﹕ onTouchEvent:true action:ACTION_MOVE
    E/MyLayout﹕ onTouchEvent:true action:ACTION_MOVE

    ......

    E/MyLayout﹕ onTouchEvent:true action:ACTION_MOVE
    E/MyLayout﹕ onTouchEvent:true action:ACTION_UP

MyLayout的down事件首先传递到onInterceptTouchEvent方法；

onInterceptTouchEvent方法返回true，down直接传递给MyLayout的onTouchEvent方法处理，不传递给MyLayout2；MyLayout2的onInterceptTouchEvent方法和onTouchEvent方法返回值不影响事件传递

MyLayout的onTouchEvent返回了true，表示MyLayout消费了down事件，不再向上传递

后续的move、up等事件直接由父ViewGroup传递给MyLayout的onTouchEvent方法处理，不在经过MyLayout的onInterceptTouchEvent方法。

###总结

1. touch事件在onInterceptTouchEvent方法中的传递由父ViewGroup到子ViewGroup，在onTouchEvent方法中传递则相反。

2. onInterceptTouchEvent方法和onTouchEvent方法的返回值为true都代表消费了事件，反之则为false

3. onInterceptTouchEvent消费事件表示将事件直接传递给ViewGroup自身的onTouchEvent事件，后续事件不再经过onInterceptTouchEvent方法；不消费事件则表示将事件传递给子View处理

4. onTouchEvent消费事件表示不再向上传递，后续事件继续传递给该View的onTouchEvent方法；不消费事件则表示将事件传递给父ViewGroup，后续事件不再传递给该View(该View是ViewGroup时onInterceptTouchEvent方法也不再收到后续事件)


***

#Demo代码

demo代码可以在我的github上下载：[Android触屏事件传递演示Demo](https://github.com/daemon369/Demo/tree/master/20140815)

***

#MyView、MyLayout、MyLayout2关键代码

{% highlight java %}
public class MyView extends View {

    private final static String TAG = "MyView";

    ...

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        boolean result = super.onTouchEvent(event);
        // result = true;
        Log.e(TAG, "onTouchEvent:" + result + " action:" + event.getAction());
        return result;
    }
}
{% endhighlight %}

{% highlight java %}
public class MyLayout extends FrameLayout {

    private final static String TAG = "MyLayout";

    ...

    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        boolean r = super.onInterceptTouchEvent(ev);
        // r = true;
        Log.e(TAG, "onInterceptTouchEvent:" + r + " action:" + ev.getAction());
        return r;
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        boolean r = super.onTouchEvent(event);
        r = true;
        Log.e(TAG, "onTouchEvent:" + r + " action:" + event.getAction());
        return r;
    }
}
{% endhighlight %}

{% highlight java %}
public class MyLayout2 extends FrameLayout {

    private final static String TAG = "MyLayout2";

    ...

    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        boolean r = super.onInterceptTouchEvent(ev);
        // r = true;
        Log.e(TAG, "onInterceptTouchEvent:" + r + " action:" + ev.getAction());
        return r;
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        boolean r = super.onTouchEvent(event);
        // r = true;
        Log.e(TAG, "onTouchEvent:" + r + " action:" + event.getAction());
        return r;
    }
}
{% endhighlight %}

***

#相关文章

[Android Touch事件传递机制(二) -- OnTouchListener & OnClickListener & OnLongClickListener][1]

[Android Touch事件传递机制(三) -- Touch事件分发(dispatchTouchEvent)][2]

***

#参考

[onInterceptTouchEvent和onTouchEvent调用时序](http://blog.csdn.net/ddna/article/details/5473293)

[1]: {% post_url 2014-08-25-android-OnTouchListener-OnClickListener-OnLongClickListener %} "Android Touch事件传递机制(二) -- OnTouchListener & OnClickListener & OnLongClickListener"

[2]: {% post_url 2014-09-11-android-dispatchTouchEvent %} "Android Touch事件传递机制(三) -- Touch事件分发(dispatchTouchEvent)"