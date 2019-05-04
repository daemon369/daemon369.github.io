---
layout: post
keywords: android, ListView, onItemClick, descendantFocusability
description: jekyll using excerpt in post
title: "ListView的onItemClick事件无法响应问题"
categories: [Android]
tags: [Android, ListView]
group: Android
icon: file-alt
date: 2014-03-18 00:00:00
---

我们在使用ListView时，可以会遇到点击列表项时，onItemClick事件没有被调用，我们的点击没有响应，这种情况往往是由于列表项点击事件被列表项中的其他控件截获的原因：

<!--excerpt-->

***

# [<b>android:descendantFocusability</b>](http://developer.android.com/reference/android/view/ViewGroup.html#attr_android:descendantFocusability)

Android官方文档中对android:descendantFocusability有以下描述：

Defines the relationship between the ViewGroup and its descendants when looking for a View to take focus.

Must be one of the following constant values.

<table>
    <tr>
        <th>Constant</th>
        <th>Value</th>
        <th>	Description</th>
    </tr>
    <tr>
        <td>beforeDescendants</td>
        <td>	0</td>
        <td>	The ViewGroup will get focus before any of its descendants.</td>
    </tr>
    <tr>
        <td>afterDescendants</td>
        <td>	1</td>
        <td>	The ViewGroup will get focus only if none of its descendants want it.</td>
    </tr>
    <tr>
        <td>blocksDescendants</td>
        <td>2</td>
        <td>The ViewGroup will block its descendants from receiving focus.</td>
    </tr>
</table>


beforeDescendants：ViewGroup将在其子控件之前获取焦点

afterDescendants：只有当子控件没有获取焦点时，ViewGroup才会获取到焦点

blocksDescendants：ViewGroup将会阻塞其子控件获取焦点

有如下的ListView的列表项item布局：

    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="50dp"
        android:orientation="horizontal">
    
        <TextView
            android:id="@+id/text"
            android:layout_height="wrap_content"
            android:layout_width="wrap_content"
            android:text="abcdefg" />
        
        <Button
            android:id="@+id/button"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="点击" />
    </LinearLayout>

我们希望ListView的item的子控件和item自身能够分别响应不同的事件，例如点击item和Button分别有不同的响应：

    listView.setOnItemClickListener(new OnItemClickListener() {
        @Override
        public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
            //item的点击事件
        }
    });
    
    ...
    
    button.setOnClickListener(new View.OnClickListener() {
    
        @Override
        public void onClick(View v) {
            //item子控件点击事件
        }
    });

但是结果常常是item的onItemClick事件无法响应。这是由于item的焦点被子控件获取了，因此需要使子控件不获取焦点，并将item的android:descendantFocusability属性设置为blocksDescendants。

item.xml

    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="50dp"
        android:orientation="horizontal"
        android:descendantFocusability="blocksDescendants">
        
        <TextView
            android:id="@+id/text"
            android:layout_height="wrap_content"
            android:layout_width="wrap_content"
            android:text="abcdefg" />
        
        <Button
            android:id="@+id/button"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:focusable="false"
            android:text="点击" />
    </LinearLayout>



其他的控件例如CheckBox、ImageButton等都是类似的。
