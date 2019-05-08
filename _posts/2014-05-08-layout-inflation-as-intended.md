---
layout: post
keywords: android, layout inflater
description: layout inflation as intended
title: "Android:LayoutInflater分析"
categories: [Android]
tags: [android, LayoutInflater, inflate]
group: Android
icon: file-alt
date: 2014-05-08 00:00:00
---

在Android中，我们使用LayoutInflater来解析布局xml文件并生成对应的View对象的继承树。LayoutInflater提供了两个常用的方法：

    1. inflate(int resource, ViewGroup root);
    2. inflate(int resource, ViewGroup root, boolean attachToRoot);

我们经常使用第一个函数来解析布局文件：

    inflater.inflate(R.layout.my_layout, null);

但是，你可能没有意识到，这样使用常常是不恰当的，甚至是错误的，使用这种方法往往实现不了我们预期的显示效果。

<!--excerpt-->

***

# LayoutInflater

首先来了解下LayoutInflater的工作方式：

inflate方法共提供了三个参数，其中第一个参数是我们需要解析的布局资源ID，第二个参数是我们希望解析的布局资源关联的root View，第三个参数决定了我们的布局资源解析后是否关联到第二个参数传入的root上。

根据源码可知，第一个inflater方法实际上也是调用第二个方法：

    public View inflate(int resource, ViewGroup root) {
        return inflate(resource, root, root != null);
    }

当我们不需要我们的自定义布局直接attach到root view上时，我们经常使用第一个方法并传入root为null，并认为下面的调用是等价的：

    inflater.inflate(R.layout.my_layout, null);
    inflater.inflate(R.layout.my_layout, root, null);

但二者其实是有区别的。

分析源码可知，所有的inflate方法最终都会调用以下方法：

    public View inflate(XmlPullParser parser, ViewGroup root, boolean attachToRoot) {
        synchronized (mConstructorArgs) {
            final AttributeSet attrs = Xml.asAttributeSet(parser);
            ...
            View result = root;

            try {
                ...

                if (TAG_MERGE.equals(name)) {
                    if (root == null || !attachToRoot) {
                        throw new InflateException("<merge /> can be used only with a valid "
                                + "ViewGroup root and attachToRoot=true");
                    }

                    rInflate(parser, root, attrs, false);
                } else {
                    // Temp is the root view that was found in the xml
                    View temp;
                    if (TAG_1995.equals(name)) {
                        temp = new BlinkLayout(mContext, attrs);
                    } else {
                        temp = createViewFromTag(root, name, attrs);
                    }

                    ViewGroup.LayoutParams params = null;

                    if (root != null) {
                        if (DEBUG) {
                            System.out.println("Creating params from root: " +
                                    root);
                        }
                        // Create layout params that match root, if supplied
                        params = root.generateLayoutParams(attrs);
                        if (!attachToRoot) {
                            // Set the layout params for temp if we are not
                            // attaching. (If we are, we use addView, below)
                            temp.setLayoutParams(params);
                        }
                    }

                    // Inflate all children under temp
                    rInflate(parser, temp, attrs, true);

                    // We are supposed to attach all the views we found (int temp)
                    // to root. Do that now.
                    if (root != null && attachToRoot) {
                        root.addView(temp, params);
                    }

                    // Decide whether to return the root that was passed in or the
                    // top view found in xml.
                    if (root == null || !attachToRoot) {
                        result = temp;
                    }
                }

            } catch (XmlPullParserException e) {
                ...
            } catch (IOException e) {
                ...
            } finally {
                ...
            }

            return result;
        }
    }

可以看到，首先调用createViewFromTag解析布局文件生成View实例temp。

当root非空时，会调用generateLayoutParams生成LayoutParams对象的实例params。

如果attachToRoot为true，则调用root.addView(temp, params)添加解析的temp到root并返回root；如果attachToRoot为false，则调用temp.setLayoutParams(params)设置temp的LayoutParams属性。

而如果root为空时，则返回解析生成的temp。

这就可以看出root view的作用了，即使不需要将temp实例attach到root，root仍然可以用来生成LayoutParams实例在temp的measure过程中提供temp大小估算的依据。

***

# 案例分析

我们经常使用ListView并实现自定义的adapter实例，在adapter中的getView方法中我们提供自定义的列表项布局，这就需要使用LayoutInflater来解析布局文件：

    getView(int position, View convertView, ViewGroup parent);

类似的，在fragment中，我们经常需要在onCreateView方法中解析自定义的布局文件。

    onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedinstanceState);

可以看到，这两者都传入了父ViewGroup。

但是如果我们使用LayoutInflater解析布局文件时自动关联解析的View到root，程序就会抛出异常。

那么为什么这里传入了父ViewGroup却不允许我们关联父ViewGroup？其实从上面的源码分析就可以看出，inflater传入的root是有助于自定义View的measure过程的。我们知道，一个View的LayoutParams只有在父布局中才有意义，它在measure的过程中会根据父布局的LayoutParams以及自身设置的LayoutParams来估算自己最终的大小。

我们可能会遇到这种情况，我们在ListView的item布局文件中设置了item的LayoutParams，即设置了类似android:layout_xxx的属性，但是在inflate时我们没有传入root，结果我们设置的LayoutParams没有起作用，root为null时，解析出来的View继承树中顶层的View因为没有父布局，系统会抛弃我们在布局文件的顶层元素中设置的LayoutParams属性，产生一个默认的LayoutParams设置到解析的View中。这就是我们设置的item大小常常不符合预期的原因。

***

# 程序示例

列表项布局R.layout.list_item：

    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="?android:attr/listPreferredItemHeight"
        android:orientation="vertical"
        android:background="@android:color/white" >
        <TextView 
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="列表项"/>
    </LinearLayout>

我们给列表项设置了固定大小的高度，然后在getView方法中inflate布局：

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        if (null == convertView) {
            convertView = LayoutInflater.from(MainActivity.this).inflate(R.layout.list_item,
                    null);
        }
        return convertView;
    }

看看效果：

![](/image/2014-05-08/1.png)

很明显这不是我们期望的效果。那么将使用父ViewGroup后是什么效果：

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        if (null == convertView) {
            convertView = LayoutInflater.from(MainActivity.this).inflate(R.layout.list_item,
                    parent, false);
        }
        return convertView;
    }

结果为：

![](/image/2014-05-08/2.png)

这才是我们期望的结果。

***

# 例外的情况

一般我们不建议使用第一个inflate方法传入root为null的使用方式，但是也有确实需要传入null的使用场景。

当我们使用AlertDialog并实现自定义的布局时，就需要这样使用：

    AlertDialog.Builder builder = new AlertDialog.Builder(context);
    View content = LayoutInflater.from(context).inflate(R.layout.item_row, null);
    builder.setTitle("My Dialog");
    builder.setView(content);
    builder.setPositiveButton("OK", null);
    builder.show();

AlertDialog.Builder支持自定义布局，但只提供了如下的设置布局的方法：

    public Builder setView(View view);

因此我们需要自己来inflate布局文件。而dialog并没有提供一个root view，因此我们无法使用一个已有的父ViewGroup来为我们自定义的布局提供measure的依据，所以AlertDialog会忽略我们自定义布局设置的LayoutParams，而是使用match_parent。

***

# 参考:

[Layout Inflation as Intended](http://www.doubleencore.com/2013/05/layout-inflation-as-intended/)
