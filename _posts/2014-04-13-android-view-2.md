---
layout: post
keywords: android, view
description: android view study
title: "Android View学习（二） -- View绘制流程"
categories: [Android]
tags: [android, View]
group: Android
icon: file-alt
date: 2014-04-13 01:00:00
---

    基于 android 2.3.3_r1 代码研究

Android中的任何一个布局、任何一个控件其实都是直接或间接继承自View的，如 TextView、Button、ImageView、ListView 等。知道了View的绘制原理，我们就可以自由定制个性化的View。

每一个视图的绘制过程都必须经历三个最主要的阶段，即 onMeasure()、onLayout() 和 onDraw()，下面我们逐个对这三个阶段展开进行探讨。

***

# 一. onMeasure

measure 即测量的意思，onMeasure 方法顾名思义就是用于测量视图的大小的。View 系统的绘制流程会从 ViewRoot 的 performTraversals() 方法中开始，在其内部调用 View 的 measure() 方法。measure() 方法接收两个参数，widthMeasureSpec 和 heightMeasureSpec，这两个值分别用于确定视图的宽度和高度的规格和大小。

<!--excerpt-->

## MeasureSpec

{% highlight java linenos %}
public static class MeasureSpec {
    private static final int MODE_SHIFT = 30;
    private static final int MODE_MASK = 0x3 << MODE_SHIFT;

    /**
     * Measure specification mode: The parent has not imposed any constraint
     * on the child. It can be whatever size it wants.
     */
    public static final int UNSPECIFIED = 0 << MODE_SHIFT;

    /**
     * Measure specification mode: The parent has determined an exact size
     * for the child. The child is going to be given those bounds regardless
     * of how big it wants to be.
     */
    public static final int EXACTLY = 1 << MODE_SHIFT;

    /**
     * Measure specification mode: The child can be as large as it wants up
     * to the specified size.
     */
    public static final int AT_MOST = 2 << MODE_SHIFT;

    //根据size和mode组装MeasureSpec
    public static int makeMeasureSpec(int size, int mode) {
        return size + mode;
    }

    //获取View的mode
    public static int getMode(int measureSpec) {
        return (measureSpec & MODE_MASK);
    }

    //获取View的size
    public static int getSize(int measureSpec) {
        return (measureSpec & ~MODE_MASK);
    }
}
{% endhighlight %}

MeasureSpec 使用一个 int 类型来存储 View 的模式 (specMode) 和大小 (specSize)，其中前两位代表模式，后面30位代表 View 的大小。specMode 一共有三种类型，如下所示：

### 1. EXACTLY

表示父视图希望子视图的大小应该是由 specSize 的值来决定的，系统默认会按照这个规则来设置子视图的大小，开发人员也可以按照自己的意愿设置成任意的大小。

### 2. AT_MOST

表示子视图最多只能是 specSize 中指定的大小，开发人员应该尽可能小得去设置这个视图，并且保证不会超过 specSize。系统默认会按照这个规则来设置子视图的大小，开发人员也可以按照自己的意愿设置成任意的大小。

### 3. UNSPECIFIED

表示开发人员可以将视图按照自己的意愿设置成任意的大小，没有任何限制。这种情况比较少见，不太会用到。

那么 widthMeasureSpec 和 heightMeasureSpec 这两个值又是从哪里得到的呢？通常情况下，这两个值都是由父视图经过计算后传递给子视图的，说明父视图会在一定程度上决定子视图的大小。但是最外层的根视图，它的 widthMeasureSpec 和 heightMeasureSpec 又是从哪里得到的呢？这就需要去分析 ViewRoot 中的源码了，观察 performTraversals() 方法可以发现如下代码：

{% highlight java %}
    childWidthMeasureSpec = getRootMeasureSpec(desiredWindowWidth, lp.width);  
    childHeightMeasureSpec = getRootMeasureSpec(desiredWindowHeight, lp.height); 
{% endhighlight %}

可以看到，这里调用了 getRootMeasureSpec() 方法去获取 widthMeasureSpec 和 heightMeasureSpec 的值，注意方法中传入的参数，其中 lp.width 和 lp.height 在创建 ViewGroup 实例的时候就被赋值了，它们都等于 MATCH_PARENT。然后看下 getRootMeasureSpec() 方法中的代码，如下所示：

{% highlight java linenos %}
private int getRootMeasureSpec(int windowSize, int rootDimension) {
    int measureSpec;  
    switch (rootDimension) {  
    case ViewGroup.LayoutParams.MATCH_PARENT:  
        measureSpec = MeasureSpec.makeMeasureSpec(windowSize, MeasureSpec.EXACTLY);  
        break;  
    case ViewGroup.LayoutParams.WRAP_CONTENT:  
        measureSpec = MeasureSpec.makeMeasureSpec(windowSize, MeasureSpec.AT_MOST);  
        break;  
    default:  
        measureSpec = MeasureSpec.makeMeasureSpec(rootDimension, MeasureSpec.EXACTLY);  
        break;  
    }  
    return measureSpec;  
}
{% endhighlight %}

可以看到，这里使用了 MeasureSpec.makeMeasureSpec() 方法来组装一个 MeasureSpec，当 rootDimension 参数等于 MATCH_PARENT 的时候，MeasureSpec 的 specMode 就等于 EXACTLY，当 rootDimension 等于 WRAP_CONTENT 的时候，MeasureSpec 的 specMode 就等于 AT_MOST。并且 MATCH_PARENT 和 WRAP_CONTENT 时的 specSize 都是等于 windowSize 的，也就意味着根视图总是会充满全屏的。

## measure()

介绍了这么多 MeasureSpec 相关的内容，接下来我们看下 View 的 measure() 方法里面的代码吧，如下所示：

{% highlight java linenos %}
public final void measure(int widthMeasureSpec, int heightMeasureSpec) {
    if ((mPrivateFlags & FORCE_LAYOUT) == FORCE_LAYOUT ||  
            widthMeasureSpec != mOldWidthMeasureSpec ||  
            heightMeasureSpec != mOldHeightMeasureSpec) {  
        mPrivateFlags &= ~MEASURED_DIMENSION_SET;  

        onMeasure(widthMeasureSpec, heightMeasureSpec);  
        if ((mPrivateFlags & MEASURED_DIMENSION_SET) != MEASURED_DIMENSION_SET) {  
            throw new IllegalStateException("onMeasure() did not set the"  
                    + " measured dimension by calling"  
                    + " setMeasuredDimension()");  
        }
        mPrivateFlags |= LAYOUT_REQUIRED;
    }
    mOldWidthMeasureSpec = widthMeasureSpec;  
    mOldHeightMeasureSpec = heightMeasureSpec;  
}
{% endhighlight %}

注意，measure() 这个方法是 final 的，因此我们无法在子类中去重写这个方法。

第7行调用了 onMeasure() 方法，这里才是真正去测量并设置 View 大小的地方:

{% highlight java linenos %}
/**
 * 这个方法将会被 {@link #measure(int, int)} 方法调用，子类覆盖此方法以提供其内容的尺寸的精确测量
 *
 * 子类覆盖此方法时，必须调用 {@link #setMeasuredDimension(int, int)} 方法来存储测算后的内容尺寸，否则
 * {@link #measure(int, int)} 方法会抛出 IllegalStateException 异常
 *
 * View 类提供的默认尺寸测算方法，内容尺寸大小等于背景的尺寸大小
 *
 * 子类覆盖此方法，需要注意保证测算的宽、高大于等于 ({@link #getSuggestedMinimumHeight()} 和
 * {@link #getSuggestedMinimumWidth()}) 方法提供的最小宽、高
 */
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
            getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
}
{% endhighlight %}

默认会调用 getDefaultSize() 方法来获取视图的大小：

{% highlight java linenos %}
public static int getDefaultSize(int size, int measureSpec) {
    int result = size;  
    int specMode = MeasureSpec.getMode(measureSpec);  
    int specSize = MeasureSpec.getSize(measureSpec);  
    switch (specMode) {  
    case MeasureSpec.UNSPECIFIED:  
        result = size;  
        break;  
    case MeasureSpec.AT_MOST:  
    case MeasureSpec.EXACTLY:  
        result = specSize;  
        break;  
    }  
    return result;  
}
{% endhighlight %}

这里传入的 measureSpec 是从 measure() 方法中传递过来的。然后调用 MeasureSpec.getMode() 获取 specMode，调用 MeasureSpec.getSize() 获取 specSize。接下来进行判断，如果 specMode 等于 AT_MOST 或 EXACTLY 就返回 specSize，这也是系统默认的行为。之后会在 onMeasure() 方法中调用 setMeasuredDimension() 方法来设定测量出的大小，这样一次 measure 过程就结束了。

当然，一个界面的展示可能会涉及到很多次的 measure，因为一个布局中一般都会包含多个子视图，每个视图都需要经历一次 measure 过程。ViewGroup 中定义了一个 measureChildren() 方法来测量子视图的大小，如下所示：

{% highlight java linenos %}
protected void measureChildren(int widthMeasureSpec, int heightMeasureSpec) {
    final int size = mChildrenCount;  
    final View[] children = mChildren;  
    for (int i = 0; i < size; ++i) {  
        final View child = children[i];  
        if ((child.mViewFlags & VISIBILITY_MASK) != GONE) {  
            measureChild(child, widthMeasureSpec, heightMeasureSpec);  
        }  
    }  
}
{% endhighlight %}

这里首先会去遍历当前布局下的所有子视图，然后逐个调用 measureChild() 方法来测量相应子视图的大小：

{% highlight java linenos %}
protected void measureChild(View child, int parentWidthMeasureSpec,  
        int parentHeightMeasureSpec) {  
    final LayoutParams lp = child.getLayoutParams();  
    final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,  
            mPaddingLeft + mPaddingRight, lp.width);  
    final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,  
            mPaddingTop + mPaddingBottom, lp.height);  
    child.measure(childWidthMeasureSpec, childHeightMeasureSpec);  
}
{% endhighlight %}

可以看到，在第4行和第6行分别调用了 getChildMeasureSpec() 方法来去计算子视图的 MeasureSpec，计算的依据就是布局文件中定义的 MATCH_PARENT、WRAP_CONTENT 等值，这个方法的内部细节就不再贴出。然后在第8行调用子视图的 measure() 方法，并把计算出的 MeasureSpec 传递进去，之后的流程就和前面所介绍的一样了。

当然，onMeasure()方法是可以重写的，也就是说，如果你不想使用系统默认的测量方式，可以按照自己的意愿进行定制，比如：

{% highlight java linenos %}
public class MyView extends View {  
    ......  

    @Override  
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {  
        setMeasuredDimension(200, 200);  
    }    
}
{% endhighlight %}

这样的话就把 View 默认的测量流程覆盖掉了，不管在布局文件中定义 MyView 这个视图的大小是多少，最终在界面上显示的大小都将会是200*200。

需要注意的是，在 setMeasuredDimension() 方法调用之后，我们才能使用 getMeasuredWidth() 和 getMeasuredHeight() 来获取视图测量出的宽高，以此之前调用这两个方法得到的值都会是0。

由此可见，View 大小的控制是由父 View、布局文件、以及 View 本身共同完成的，父 View 会提供给子 View 参考的大小，而开发人员可以在 XML 文件中指定 View 的大小，然后 View 本身会对最终的大小进行拍板。

到此为止，我们就把 View 绘制流程的第一阶段分析完了。

***

# 二. onLayout()

measure 过程结束后，视图的大小就已经测量好了，接下来就是 layout 的过程了。正如其名字所描述的一样，这个方法是用于给视图进行布局的，也就是确定视图的位置。ViewRoot 的 performTraversals() 方法会在 measure 结束后继续执行，并调用 View 的 layout() 方法来执行此过程：

{% highlight java %}
host.layout(0, 0, host.mMeasuredWidth, host.mMeasuredHeight); 
{% endhighlight %}

layout()方法接收四个参数，分别代表着左、上、右、下的坐标，当然这个坐标是相对于当前视图的父视图而言的。可以看到，这里还把刚才测量出的宽度和高度传到了layout()方法中。那么我们来看下layout()方法中的代码是什么样的吧，如下所示：

{% highlight java linenos %}
/**
 * layout是 view 布局机制的第二阶段，父 view 调用所有子 view 的 layout 方法，
 * 根据 measure 过程中测算的尺寸数据，指定子 view 的位置及大小
 */
public final void layout(int l, int t, int r, int b) {
    boolean changed = setFrame(l, t, r, b);
    if (changed || (mPrivateFlags & LAYOUT_REQUIRED) == LAYOUT_REQUIRED) {

        onLayout(changed, l, t, r, b);
        mPrivateFlags &= ~LAYOUT_REQUIRED;
    }
    mPrivateFlags &= ~FORCE_LAYOUT;
}
{% endhighlight %}

在layout()方法中，首先会调用setFrame()方法来判断视图的大小是否发生过变化，以确定有没有必要对当前的视图进行重绘，同时还会在这里把传递过来的四个参数分别赋值给mLeft、mTop、mRigh t和 mBottom 这几个变量。接下来会在第9行调用 onLayout() 方法。View 中的 onLayout() 方法是一个空方法，因为 onLayout() 过程是为了确定视图在布局中所在的位置，而这个操作应该是由布局来完成的，即父视图决定子视图的显示位置。既然如此，我们来看下 ViewGroup 中的 onLayout() 方法是怎么写的吧，代码如下：

{% highlight java %}
@Override
protected abstract void onLayout(boolean changed, int l, int t, int r, int b);
{% endhighlight %}

可以看到，ViewGroup 中的 onLayout() 方法竟然是一个抽象方法，这就意味着所有 ViewGroup 的子类都必须重写这个方法。没错，像 LinearLayout、RelativeLayout 等布局，都是重写了这个方法，然后在内部按照各自的规则对子视图进行布局的。由于 LinearLayout 和 RelativeLayout 的布局规则都比较复杂，就不单独拿出来进行分析了，这里我们尝试自定义一个布局，借此来更深刻地理解 onLayout() 的过程。

自定义的这个布局目标很简单，只要能够包含一个子视图，并且让子视图正常显示出来就可以了：

{% highlight java linenos %}
public class SimpleLayout extends ViewGroup {  
  
    public SimpleLayout(Context context, AttributeSet attrs) {  
        super(context, attrs);  
    }  
  
    @Override  
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {  
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);  
        if (getChildCount() > 0) {  
            View childView = getChildAt(0);  
            measureChild(childView, widthMeasureSpec, heightMeasureSpec);  
        }  
    }  
  
    @Override  
    protected void onLayout(boolean changed, int l, int t, int r, int b) {  
        if (getChildCount() > 0) {  
            View childView = getChildAt(0);  
            childView.layout(0, 0, childView.getMeasuredWidth(), childView.getMeasuredHeight());  
        }  
    }
}
{% endhighlight %}

代码非常的简单，我们来看下具体的逻辑吧。你已经知道，onMeasure() 方法会在 onLayout() 方法之前调用，因此这里在 onMeasure() 方法中判断 SimpleLayout 中是否有包含一个子视图，如果有的话就调用 measureChild() 方法来测量出子视图的大小。

接着在 onLayout() 方法中同样判断 SimpleLayout 是否有包含一个子视图，然后调用这个子视图的 layout() 方法来确定它在 SimpleLayout 布局中的位置，这里传入的四个参数依次是0、0、childView.getMeasuredWidth() 和 childView.getMeasuredHeight()，分别代表着子视图在 SimpleLayout 中左上右下四个点的坐标。其中，调用 childView.getMeasuredWidth() 和 childView.getMeasuredHeight() 方法得到的值就是在 onMeasure() 方法中测量出的宽和高。

这样就已经把 SimpleLayout 这个布局定义好了，下面在 XML 文件中使用它：

{% highlight java linenos %}
<com.example.viewtest.SimpleLayout xmlns:android="http://schemas.android.com/apk/res/android"  
    android:layout_width="match_parent"  
    android:layout_height="match_parent" >  

    <ImageView   
        android:layout_width="wrap_content"  
        android:layout_height="wrap_content"  
        android:src="@drawable/ic_launcher" />  

</com.example.viewtest.SimpleLayout>
{% endhighlight %}

可以看到，我们能够像使用普通的布局文件一样使用 SimpleLayout，只是注意它只能包含一个子视图，多余的子视图会被舍弃掉。这里 SimpleLayout 中包含了一个 ImageView，并且 ImageView 的宽高都是 wrap_content。现在运行一下程序，结果如下图所示：

![](/image/2014-04-13/3.png)

OK！ImageView 成功已经显示出来了，并且显示的位置也正是我们所期望的。如果你想改变 ImageView 显示的位置，只需要改变 childView.layout() 方法的四个参数就行了。

在 onLayout() 过程结束后，我们就可以调用 getWidth() 方法和 getHeight() 方法来获取视图的宽高了。说到这里，我相信很多朋友长久以来都会有一个疑问，getWidth() 方法和 getMeasureWidth() 方法到底有什么区别呢？它们的值好像永远都是相同的。其实它们的值之所以会相同基本都是因为布局设计者的编码习惯非常好，实际上它们之间的差别还是挺大的。

首先 getMeasureWidth() 方法在 measure() 过程结束后就可以获取到了，而 getWidth() 方法要在 layout() 过程结束后才能获取到。另外，getMeasureWidth() 方法中的值是通过 setMeasuredDimension() 方法来进行设置的，而 getWidth() 方法中的值则是通过视图右边的坐标减去左边的坐标计算出来的。

观察 SimpleLayout 中 onLayout() 方法的代码，这里给子视图的 layout() 方法传入的四个参数分别是0、0、childView.getMeasuredWidth() 和 childView.getMeasuredHeight()，因此 getWidth() 方法得到的值就是 childView.getMeasuredWidth() - 0 = childView.getMeasuredWidth()，所以此时 getWidth() 方法和 getMeasuredWidth() 得到的值就是相同的，但如果你将 onLayout() 方法中的代码进行如下修改：

{% highlight java linenos %}
@Override
protected void onLayout(boolean changed, int l, int t, int r, int b) {  
    if (getChildCount() > 0) {  
        View childView = getChildAt(0);  
        childView.layout(0, 0, 200, 200);  
    }  
}
{% endhighlight %}

这样 getWidth() 方法得到的值就是200 - 0 = 200，不会再和 getMeasuredWidth() 的值相同了。当然这种做法充分不尊重 measure() 过程计算出的结果，通常情况下是不推荐这么写的。getHeight() 与 getMeasureHeight() 方法之间的关系同上，就不再重复分析了。

到此为止，我们把视图绘制流程的第二阶段也分析完了。

***

# 三. onDraw()

measure 和 layout 的过程都结束后，接下来就进入到 draw 的过程了，这里才真正地开始对视图进行绘制。ViewRoot 中的代码会继续执行并创建出一个 Canvas 对象，然后调用 View 的 draw() 方法来执行具体的绘制工作。

draw() 方法内部的绘制过程总共可以分为六步:

     1. Draw the background    绘制背景
     2. If necessary, save the canvas' layers to prepare for fading
     3. Draw view's content      绘制 view 的内容
     4. Draw children                 绘制子 view
     5. If necessary, draw the fading edges and restore layers
     6. Draw decorations (scrollbars for instance) 绘制滚动条等装饰

其中第二步和第五步在一般情况下很少用到，因此这里我们只分析简化后的绘制过程。代码如下所示：

{% highlight java linenos %}
public void draw(Canvas canvas) {
    final int privateFlags = mPrivateFlags;
    final boolean dirtyOpaque = (privateFlags & DIRTY_MASK) == DIRTY_OPAQUE &&
            (mAttachInfo == null || !mAttachInfo.mIgnoreDirtyState);
    mPrivateFlags = (privateFlags & ~DIRTY_MASK) | DRAWN;

    // Step 1, draw the background, if needed
    int saveCount;

    if (!dirtyOpaque) {
        final Drawable background = mBGDrawable;
        if (background != null) {
            final int scrollX = mScrollX;
            final int scrollY = mScrollY;

            if (mBackgroundSizeChanged) {
                background.setBounds(0, 0,  mRight - mLeft, mBottom - mTop);
                mBackgroundSizeChanged = false;
            }

            if ((scrollX | scrollY) == 0) {
                background.draw(canvas);
            } else {
                canvas.translate(scrollX, scrollY);
                background.draw(canvas);
                canvas.translate(-scrollX, -scrollY);
            }
        }
    }

    // skip step 2 & 5 if possible (common case)
    final int viewFlags = mViewFlags;
    boolean horizontalEdges = (viewFlags & FADING_EDGE_HORIZONTAL) != 0;
    boolean verticalEdges = (viewFlags & FADING_EDGE_VERTICAL) != 0;
    if (!verticalEdges && !horizontalEdges) {
        // Step 3, draw the content
        if (!dirtyOpaque) onDraw(canvas);

        // Step 4, draw the children
        dispatchDraw(canvas);

        // Step 6, draw decorations (scrollbars)
        onDrawScrollBars(canvas);

        // we're done...
        return;
    }
}
{% endhighlight %}

可以看到，第一步是从第7行代码开始的，这一步的作用是对视图的背景进行绘制。这里会先得到一个 mBGDrawable 对象，然后根据 layout 过程确定的视图位置来设置背景的绘制区域，之后再调用 Drawable 的 draw() 方法来完成背景的绘制工作。那么这个 mBGDrawable 对象是从哪里来的呢？其实就是在 XML 中通过 android:background 属性设置的图片或颜色。当然你也可以在代码中通过 setBackgroundColor()、setBackgroundResource() 等方法进行赋值。

接下来的第三步的作用是对视图的内容进行绘制。可以看到，这里去调用了一下 onDraw() 方法，那么 onDraw() 方法里又写了什么代码呢？进去一看你会发现，原来又是个空方法啊。其实也可以理解，因为每个视图的内容部分肯定都是各不相同的，这部分的功能交给子类来去实现也是理所当然的。

第三步完成之后紧接着会执行第四步，这一步的作用是对当前视图的所有子视图进行绘制。但如果当前的视图没有子视图，那么也就不需要进行绘制了。因此你会发现 View 中的 dispatchDraw() 方法又是一个空方法，而 ViewGroup 的 dispatchDraw() 方法中就会有具体的绘制代码。

以上都执行完后就会进入到第六步，也是最后一步，这一步的作用是对视图的滚动条进行绘制。那么你可能会奇怪，当前的视图又不一定是 ListView 或者 ScrollView，为什么要绘制滚动条呢？其实不管是 Button 也好，TextView 也好，任何一个视图都是有滚动条的，只是一般情况下我们都没有让它显示出来而已。绘制滚动条的代码逻辑也比较复杂，这里就不再贴出来了，因为我们的重点是第三步过程。

通过以上流程分析，相信大家已经知道，View 是不会帮我们绘制内容部分的，因此需要每个视图根据想要展示的内容来自行绘制。如果你去观察 TextView、ImageView 等类的源码，你会发现它们都有重写 onDraw() 这个方法，并且在里面执行了相当不少的绘制逻辑。绘制的方式主要是借助 Canvas 这个类，它会作为参数传入到 onDraw() 方法中，供给每个视图使用。Canvas 这个类的用法非常丰富，基本可以把它当成一块画布，在上面绘制任意的东西，那么我们就来尝试一下吧。

这里简单起见，我只是创建一个非常简单的视图，并且用 Canvas 随便绘制了一点东西，代码如下所示：

{% highlight java linenos %}
public class MyView extends View {
  
    private Paint mPaint;  
  
    public MyView(Context context, AttributeSet attrs) {  
        super(context, attrs);  
        mPaint = new Paint(Paint.ANTI_ALIAS_FLAG);  
    }  
  
    @Override  
    protected void onDraw(Canvas canvas) {  
        mPaint.setColor(Color.YELLOW);  
        canvas.drawRect(0, 0, getWidth(), getHeight(), mPaint);  
        mPaint.setColor(Color.BLUE);  
        mPaint.setTextSize(20);  
        String text = "Hello View";  
        canvas.drawText(text, 0, getHeight() / 2, mPaint);  
    }  
}
{% endhighlight %}

可以看到，我们创建了一个自定义的 MyView 继承自 View，并在MyView的构造函数中创建了一个 Paint 对象。Paint 就像是一个画笔一样，配合着 Canvas 就可以进行绘制了。这里我们的绘制逻辑比较简单，在 onDraw() 方法中先是把画笔设置成黄色，然后调用 Canvas 的 drawRect() 方法绘制一个矩形。然后在把画笔设置成蓝色，并调整了一下文字的大小，然后调用 drawText() 方法绘制了一段文字。

就这么简单，一个自定义的视图就已经写好了，现在可以在 XML 中加入这个视图：

{% highlight java linenos %}
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >

    <com.example.viewtest.MyView
        android:layout_width="200dp"
        android:layout_height="100dp"
        />
</LinearLayout>
{% endhighlight %}

将 MyView 的宽度设置成200dp，高度设置成100dp，然后运行一下程序，结果如下图所示：

![](/image/2014-04-13/4.png)

图中显示的内容也正是 MyView 这个视图的内容部分了。由于我们没给 MyView 设置背景，因此这里看不出来 View 自动绘制的背景效果。

***

# 参考:

[Android视图绘制流程完全解析，带你一步步深入了解View(二)](http://blog.csdn.net/guolin_blog/article/details/16330267)
