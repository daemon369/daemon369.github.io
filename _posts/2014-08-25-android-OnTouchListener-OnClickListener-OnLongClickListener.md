---
layout: post
keywords: andrid, onTouchEvent, OnClickListener, OnLongClickListener
description: OnTouchListener & OnClickListener & OnLongClickListener
title: "Android Touch事件传递机制(二) -- OnTouchListener & OnClickListener & OnLongClickListener"
categories: [Android]
tags: [android, dispatchTouchEvent, OnTouchListener, onTouch, onTouchEvent, OnClickListener, OnLongClickListener]
group: Android
icon: file-alt
---
{% include JB/setup %}

***

(本文基于android-2.3.3_r1代码研究)

在[Android Touch事件传递机制(一) -- onInterceptTouchEvent & onTouchEvent](http://daemon369.github.io/android/2014/08/17/android-onInterceptTouchEvent-onTouchEvent/)这篇文章中研究了Android触屏事件的分发机制；本文继续深入研究。

***

#OnTouchListener & onTouchEvent

首先我们来看看View的OnTouchListener与onTouchEvent方法的区别与联系，如下布局：

{% highlight java %}
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <com.daemon.demo.touchdemo.widget.MyView
        android:id="@+id/my_view"
        android:layout_width="100dp"
        android:layout_height="100dp"
        android:background="#00ff00"/>

</RelativeLayout>
{% endhighlight %}

设置OnTouchListener：

{% highlight java %}
MyView view = (MyView) findViewById(R.id.my_view);

view.setOnTouchListener(new View.OnTouchListener() {
    @Override
    public boolean onTouch(View v, MotionEvent event) {
        boolean ret = false;
        Log.e("Test5Activity", "onTouch:" + ret + " action:" + TouchUtil.actionToString(event.getAction()));
        return ret;
    }
});
{% endhighlight %}

运行demo，在MyView上按下触屏并滑动后松开，查看log：

    E/Test5Activity﹕ onTouch:false action:ACTION_DOWN
    E/MyView﹕ onTouchEvent:false action:ACTION_DOWN

可以看到，设置了OnTouchListener后，MyView收到down事件后首先回调OnTouchListener的onTouch方法，然后才会将down事件分发给MyView的onTouchEvent方法处理；由于onTouchEvent方法返回false，后续事件将不再分发给MyView。

onTouch方法默认返回false。那么我们返回true再运行一次试试：

    E/Test5Activity﹕ onTouch:true action:ACTION_DOWN
    E/Test5Activity﹕ onTouch:true action:ACTION_MOVE
    E/Test5Activity﹕ onTouch:true action:ACTION_MOVE

    ......

    E/Test5Activity﹕ onTouch:true action:ACTION_MOVE
    E/Test5Activity﹕ onTouch:true action:ACTION_UP

可以看到，当onTouch方法返回true时，MyView的touch事件将不再分发给其onTouchEvent方法。我们看下OnTouchListener和onTouch的解释：

{% highlight java %}
/**
 * Interface definition for a callback to be invoked when a touch event is
 * dispatched to this view. The callback will be invoked before the touch
 * event is given to the view.
 */
public interface OnTouchListener {
    /**
     * Called when a touch event is dispatched to a view. This allows listeners to
     * get a chance to respond before the target view.
     *
     * @param v The view the touch event has been dispatched to.
     * @param event The MotionEvent object containing full information about
     *        the event.
     * @return True if the listener has consumed the event, false otherwise.
     */
    boolean onTouch(View v, MotionEvent event);
}
{% endhighlight %}

设置OnTouchListener回调之后，touch事件在分发给view之后，我们将有机会在目标view之前对这个touch事件进行响应；

onTouch方法返回true表示这个touch事件已被消费掉，这就意味着touch事件不会再继续分发给目标view。

***

#OnTouchListener & OnClickListener

对MyView设置OnTouchListener 和 OnClickListener，onTouch方法返回false：

{% highlight java %}
MyView view = (MyView) findViewById(R.id.my_view);

view.setOnTouchListener(new View.OnTouchListener() {
    @Override
    public boolean onTouch(View v, MotionEvent event) {
        boolean ret;
        ret = false;
//        ret = true;
        Log.e("Test5Activity", "onTouch:" + ret + " action:" + TouchUtil.actionToString(event.getAction()));
        return ret;
    }
});

view.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        Log.e("Test5Activity", "onClick");
    }
});
{% endhighlight %}

在按钮上按下触屏、移动、放开触屏，查看log：

    E/Test5Activity﹕ onTouch:false action:ACTION_DOWN
    E/MyView﹕ onTouchEvent:true action:ACTION_DOWN
    E/Test5Activity﹕ onTouch:false action:ACTION_MOVE
    E/MyView﹕ onTouchEvent:true action:ACTION_MOVE
    E/Test5Activity﹕ onTouch:false action:ACTION_MOVE
    E/MyView﹕ onTouchEvent:true action:ACTION_MOVE

    ......

    E/Test5Activity﹕ onTouch:false action:ACTION_MOVE
    E/MyView﹕ onTouchEvent:true action:ACTION_MOVE
    E/Test5Activity﹕ onTouch:false action:ACTION_UP
    E/MyView﹕ onTouchEvent:true action:ACTION_UP
    E/Test5Activity﹕ onClick

可以看到，onTouch方法被调用了多次，分别对应了按下(ACTION_DOWN)、移动(ACTION_MOVE)、松开(ACTION_UP)等事件；onClick事件发生在ACTION_UP之后，只回调一次。

onTouch方法返回true再看下log：

{% highlight java %}
08-20 01:25:18.056: E/TouchTest(658): onTouch -> ACTION_DOWN
08-20 01:25:18.065: E/TouchTest(658): onTouch -> ACTION_MOVE
08-20 01:25:18.100: E/TouchTest(658): onTouch -> ACTION_MOVE
08-20 01:25:18.446: E/TouchTest(658): onTouch -> ACTION_MOVE
08-20 01:25:18.508: E/TouchTest(658): onTouch -> ACTION_MOVE
08-20 01:25:18.656: E/TouchTest(658): onTouch -> ACTION_UP
{% endhighlight %}

可以看到，onClick方法不再被调用。这是因为在touch事件分发给onClick方法之前，被onTouch方法拦截并消费掉，因此不再分发给onClick方法。

#OnLongClickListener

再来看看OnLongClickListener与OnTouchListener 和 OnClickListener的调用时序关系。

先看下OnLongClickListener的注释：

{% highlight java %}
/**
 * Interface definition for a callback to be invoked when a view has been clicked and held.
 */
public interface OnLongClickListener {
    /**
     * Called when a view has been clicked and held.
     *
     * @param v The view that was clicked and held.
     *
     * @return true if the callback consumed the long click, false otherwise.
     */
    boolean onLongClick(View v);
}
{% endhighlight %}

OnLongClickListener接口定义了长按触屏的回调，当我们在一个view上点击并保持一段时间后，就会唤起这个回调。onLongClick返回true表示已经消费了这次长按，否则返回false。

给MyView设置OnLongClickListener，onLongClick方法默认返回false：

{% highlight java %}
view.setOnLongClickListener(new View.OnLongClickListener() {
    @Override
    public boolean onLongClick(View v) {
        boolean ret;
        ret = false;
//        ret = true;
        Log.e("Test5Activity", "onLongClick:" + ret);
        return ret;
    }
});
{% endhighlight %}

在MyView上按下触屏并滑动，一段时间后松开触屏：

    E/Test5Activity﹕ onTouch:false action:ACTION_DOWN
    E/Test5Activity﹕ onTouch:false action:ACTION_MOVE
    E/Test5Activity﹕ onTouch:false action:ACTION_MOVE

    ......

    E/Test5Activity﹕ onTouch:false action:ACTION_MOVE
    E/Test5Activity﹕ onLongClick:false
    E/Test5Activity﹕ onTouch:false action:ACTION_MOVE

    ......

    E/Test5Activity﹕ onTouch:false action:ACTION_MOVE
    E/Test5Activity﹕ onTouch:false action:ACTION_UP
    E/Test5Activity﹕ onClick

可以看出，onClick在触屏抬起后触发；onLongClick方法在触屏抬起前触发，并需要手指按下超过一定时间；滑动手指不影响点击与长按事件的触发。

onLongClick方法返回true再看下：

    E/Test5Activity﹕ onTouch:false action:ACTION_DOWN
    E/Test5Activity﹕ onTouch:false action:ACTION_MOVE
    E/Test5Activity﹕ onTouch:false action:ACTION_MOVE

    ......

    E/Test5Activity﹕ onTouch:false action:ACTION_MOVE
    E/Test5Activity﹕ onLongClick:true
    E/Test5Activity﹕ onTouch:false action:ACTION_MOVE
    E/Test5Activity﹕ onTouch:false action:ACTION_MOVE
    
    ......
    
    E/Test5Activity﹕ onTouch:false action:ACTION_MOVE
    E/Test5Activity﹕ onTouch:false action:ACTION_UP

可以看到，onLongClick方法返回true后，onClick方法就不再触发了，这是因为onLongClick消费了长按事件。

###总结

1. 设置OnTouchListener：onTouch方法返回false时，onTouch方法及View的onTouchEvent方法依次被调用；onTouch方法返回true时，只调用onTouch方法，onTouchEvent方法不再被调用

2. 设置OnTouchListener后：onTouch方法返回false，不影响OnClickListener及OnLongClickListener的触发；onTouch方法返回true时，OnClickListener及OnLongClickListener不再触发

3. OnClickListener的触发条件是手指从触屏抬起；OnLongClickListener的触发条件是按下触屏且停留一段事件

4. onLongClick方法返回false不影响OnClickListener的触发；onLongClick方法返回true，OnClickListener不再触发

***

#Demo代码

demo代码可以在我的github上下载：[Android触屏事件传递演示Demo](https://github.com/daemon369/Demo/tree/master/20140815)

***

#参考

[onInterceptTouchEvent和onTouchEvent调用时序](http://blog.csdn.net/ddna/article/details/5473293)