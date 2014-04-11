---
layout: post
keywords: android, textview, html
description: show html text in TextView
title: "Android中使用TextView显示HTML文本"
categories: [Android]
tags: [android, TextView, html]
group: Android
icon: file-alt
---
{% include JB/setup %}

再Android中可用使用TextView来显示HTML文本：

***

#TextView中显示HTML文本：

    final String htmlString = "<font color="#00ff0000">abcdefg</font>";
    final TextView tv = (TextView)findViewById(R.id.tv);
    tv.setText(Html.fromHtml(htmlString));

如上可以显示一串红色文本。

***

#去除HTML标签

还可以将HTML文本中的标签去除，转换为普通文本：

    final String htmlString = "<font color="#00ff0000">abcdefg</font>";
    final String normalString = Html.fromHtml(htmlString).toString();
    final TextView tv = (TextView)findViewById(R.id.tv);
    tv.setText(normalString);

这样可以快捷的将HTML文本转换为不含HTML标签的文本

<!--excerpt-->

***
#参考:

[去除HTML标签](http://www.krislq.com/2012/12/android_skill_remove_the_html_tag/)