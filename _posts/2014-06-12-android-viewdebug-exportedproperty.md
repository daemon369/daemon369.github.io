---
layout: post
keywords: andrid, viewdebug, dxprotedproperty, monitor
description: user annotation
title: "@ViewDebug.ExportedProperty的使用"
categories: [Android]
tags: [android, viewdebug, exportedproperty, monitor]
group: Android
icon: file-alt
date: 2014-06-12 00:00:00
---

***

使用@ViewDebug.ExportedProperty注解，我们可以在android提供的工具Monitor(或已经废弃的DDMS)中的Hierarchy Viewer中调试View的属性。我们可以直接观察View的某个变量或方法的值，实时观察View的状态变化。

***

#[ExprotedProperty注解源码][1]

{% highlight java %}
/**
 * This annotation can be used to mark fields and methods to be dumped by
 * the view server. Only non-void methods with no arguments can be annotated
 * by this annotation.
 */
@Target({ ElementType.FIELD, ElementType.METHOD })
@Retention(RetentionPolicy.RUNTIME)
public @interface ExportedProperty {
    /**
     * When resolveId is true, and if the annotated field/method return value
     * is an int, the value is converted to an Android's resource name.
     *
     * @return true if the property's value must be transformed into an Android
     *         resource name, false otherwise
     */
    boolean resolveId() default false;
    /**
     * A mapping can be defined to map int values to specific strings. For
     * instance, View.getVisibility() returns 0, 4 or 8. However, these values
     * actually mean VISIBLE, INVISIBLE and GONE. A mapping can be used to see
     * these human readable values:
     *
     * <pre>
     * @ViewDebug.ExportedProperty(mapping = {
     *     @ViewDebug.IntToString(from = 0, to = "VISIBLE"),
     *     @ViewDebug.IntToString(from = 4, to = "INVISIBLE"),
     *     @ViewDebug.IntToString(from = 8, to = "GONE")
     * })
     * public int getVisibility() { ...
     * <pre>
     *
     * @return An array of int to String mappings
     *
     * @see android.view.ViewDebug.IntToString
     */
    IntToString[] mapping() default { };
    /**
     * A mapping can be defined to map array indices to specific strings.
     * A mapping can be used to see human readable values for the indices
     * of an array:
     *
     * <pre>
     * @ViewDebug.ExportedProperty(indexMapping = {
     *     @ViewDebug.IntToString(from = 0, to = "INVALID"),
     *     @ViewDebug.IntToString(from = 1, to = "FIRST"),
     *     @ViewDebug.IntToString(from = 2, to = "SECOND")
     * })
     * private int[] mElements;
     * <pre>
     *
     * @return An array of int to String mappings
     *
     * @see android.view.ViewDebug.IntToString
     * @see #mapping()
     */
    IntToString[] indexMapping() default { };
    /**
     * A flags mapping can be defined to map flags encoded in an integer to
     * specific strings. A mapping can be used to see human readable values
     * for the flags of an integer:
     *
     * <pre>
     * @ViewDebug.ExportedProperty(flagMapping = {
     *     @ViewDebug.FlagToString(mask = ENABLED_MASK, equals = ENABLED, name = "ENABLED"),
     *     @ViewDebug.FlagToString(mask = ENABLED_MASK, equals = DISABLED, name = "DISABLED"),
     * })
     * private int mFlags;
     * <pre>
     *
     * A specified String is output when the following is true:
     *
     * @return An array of int to String mappings
     */
    FlagToString[] flagMapping() default { };
    /**
     * When deep export is turned on, this property is not dumped. Instead, the
     * properties contained in this property are dumped. Each child property
     * is prefixed with the name of this property.
     *
     * @return true if the properties of this property should be dumped
     *
     * @see #prefix()
     */
    boolean deepExport() default false;
    /**
     * The prefix to use on child properties when deep export is enabled
     *
     * @return a prefix as a String
     *
     * @see #deepExport()
     */
    String prefix() default "";
    /**
     * Specifies the category the property falls into, such as measurement,
     * layout, drawing, etc.
     *
     * @return the category as String
     */
    String category() default "";
}
{% endhighlight %}

<!--excerpt-->

***

#使用

下面是@ViewDebug.ExportedProperty注解的部分属性的使用介绍：

##1.category

category用来指定属性的类别，例如measurement, layout, drawing等。我们在自定义View中为使用@ViewDebug.ExportedProperty注解的变量或方法指定category：

{% highlight java %}
@ExportedProperty(category = "marquee")
int x = 1;

@Override
@ExportedProperty(category = "marquee")
public boolean isFocused() {
    return true;
}
{% endhighlight %}

运行程序，在Hierarchy Viewer中查看View的属性如下：

![](/image/2014-06-12/1.png)

##2.resolveId

当resolveId为true时，如果使用注解的变量或方法的值为int数据，那么这个值会被转化为对应的Android资源的名称。

在R.java中找到一个资源ID：

    public static final int activity_main=0x7f030000;

使用注解属性resolveId：

{% highlight java %}
@ExportedProperty(category = "marquee", resolveId = true)
int b = 0x7f030000;
{% endhighlight %}

结果如下：

![](/image/2014-06-12/2.png)

##3.mapping

mapping可以将int值映射到指定的字符串值，例如View.getVisibility()返回的值是int值，View中使用注解将其映射为字符串，其中0为"VISIBLE"，4为"INVISIBLE"，8为"GONE"。我们重载View.getVisibility()并加上我们自己定制的映射：

{% highlight java %}
@Override
@ViewDebug.ExportedProperty(category = "marquee",
        mapping = {
                @ViewDebug.IntToString(from = VISIBLE, to = "MARQUEE_VISIBLE"),
                @ViewDebug.IntToString(from = INVISIBLE, to = "MARQUEE_INVISIBLE"),
                @ViewDebug.IntToString(from = GONE, to = "MARQUEE_GONE")
        })
public int getVisibility() {
    return super.getVisibility();
}
{% endhighlight %}

结果如下：

![](/image/2014-06-12/3.png)

##4.indexMapping

indexMapping可以将数组的序号映射为字符串。

{% highlight java %}
@ExportedProperty(category = "marquee",
        indexMapping = {
                @ViewDebug.IntToString(from = 0, to = "MARQUEE_FIRST"),
                @ViewDebug.IntToString(from = 1, to = "MARQUEE_SECOND"),
                @ViewDebug.IntToString(from = 2, to = "MARQUEE_THIRD")
        })
int[] elements = new int[] {
        111, 222, 333
};
{% endhighlight %}

结果如下：

![](/image/2014-06-12/4.png)

***

#Demo

{% highlight java %}
package com.daemon.demo;

import android.content.Context;
import android.graphics.Rect;
import android.util.AttributeSet;
import android.view.ViewDebug;
import android.view.ViewDebug.ExportedProperty;
import android.widget.TextView;

public class MarqueeText extends TextView {

    public MarqueeText(Context context) {
        super(context);
    }

    public MarqueeText(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    public MarqueeText(Context context, AttributeSet attrs, int defStyle) {
        super(context, attrs, defStyle);
    }

    @Override
    protected void onFocusChanged(boolean focused, int direction, Rect previouslyFocusedRect) {
    }

    @ExportedProperty(category = "marquee")
    int x = 1;

    @Override
    @ExportedProperty(category = "marquee")
    public boolean isFocused() {
        return true;
    }

    @ExportedProperty(category = "marquee", resolveId = true)
    int b = 0x7f030000;

    @Override
    @ViewDebug.ExportedProperty(category = "marquee",
            mapping = {
                    @ViewDebug.IntToString(from = VISIBLE, to = "MARQUEE_VISIBLE"),
                    @ViewDebug.IntToString(from = INVISIBLE, to = "MARQUEE_INVISIBLE"),
                    @ViewDebug.IntToString(from = GONE, to = "MARQUEE_GONE")
            })
    public int getVisibility() {
        return super.getVisibility();
    }

    @ExportedProperty(category = "marquee",
            indexMapping = {
                    @ViewDebug.IntToString(from = 0, to = "MARQUEE_FIRST"),
                    @ViewDebug.IntToString(from = 1, to = "MARQUEE_SECOND"),
                    @ViewDebug.IntToString(from = 2, to = "MARQUEE_THIRD")
            })
    int[] elements = new int[] {
            111, 222, 333
    };
}
{% endhighlight %}

***

#参考

[@ViewDebug.ExportedProperty](https://plus.google.com/+EvelioTarazonaC%C3%A1ceres/posts/fQ3G2QHaLx4)

[1]: https://android.googlesource.com/platform/frameworks/base/+/refs/heads/master/core/java/android/view/ViewDebug.java#85 "ViewDebug.java"