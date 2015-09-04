---
layout: post
keywords: android, http, httpclient, httpurlconnection
description: brief introduction of android http
title: "Android HTTP 请求：HttpClient & HttpURLConnection"
categories: [android]
tags: [android, http, httpclient, httpurlconnection]
group: Android
icon: file-alt
---
{% include JB/setup %}

Android 客户端开发中，我们通常使用 HTTP 来收发网络数据。

Android 客户端中，网络操作通常都需要如下权限：

{% highlight java linenos %}
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
{% endhighlight %}

Android 系统提供了两种 HTTP 客户端工具以供客户端开发使用：HttpURLConnection 以及 Apache 的 HttpClient。二者都支持 HTTPS、基于流的上传与下载、支持超时配置、IPv6 以及连接池等。

<!--excerpt-->

# Apache HttpClient

DefaultHttpClient 以及 AndroidHttpClient 是供 web 浏览器使用的可扩展的 HTTP 客户端工具，它们提供了大量且灵活的 API，它们的实现已经比较稳定，几乎没有 bug。大量的 API 给它们提供丰富功能的同时，也使得它们在保证兼容性的前提下很难有进一步发展空间。因此 Android 官方建议在 Android Gingerbread (Android 2.3, API Level 9)  及以上版本中使用 HttpURLConnection。

# HttpURLConnection

HttpURLConnection 是通用的轻量级的 HTTP 客户端工具，适用于大部分应用程序，且易于扩展。

在 Android Froyo 之前的版本中，存在一些 bug；尤其是在可读的输入流中调用 close()，会污染连接池。因此，需要关闭连接池以保证正常使用：

{% highlight java linenos %}
private void disableConnectionReuseIfNecessary() {
    // HTTP connection reuse which was buggy pre-froyo
    if (Integer.parseInt(Build.VERSION.SDK) < Build.VERSION_CODES.FROYO) {
        System.setProperty("http.keepAlive", "false");
    }
}
{% endhighlight %}

从 Gingerbread 版本开始，HttpURLConnection 默认使用 gzip 压缩方式发送请求，自动在 HTTP 请求头中增加：

    Accept-Encoding: gzip

相应的 HttpURLConnection 也会自动解压缩通过 getInputStream() 获取的响应数据，并将 HTTP 响应头中的 Content-Encoding 以及 Content-Length 字段清除。

使用默认 gzip 压缩方式时，getContentLength() 返回的就是压缩后的数据长度，这样响应数据就不能通过 getContentLength() 来判断数据传输是否结束，因为通过 getInputStream() 获取的数据是 HttpURLConnection 自动解压后的数据，其长度大于等于 getContentLength() 获取的值。我们可以通过 InputStream.read() 的返回值来判断传输是否结束，-1代表数据已经传输完成。

如果我们不需要默认的 gzip 压缩方式，可以通过在 HTTP 请求头中设置可用的编码方式来禁用默认的 gzip 压缩方式：

{% highlight java linenos %}
urlConnection.setRequestProperty("Accept-Encoding", "identity");
{% endhighlight %}

显式设置请求编码格式，就会禁用默认的 gzip 压缩方式，同时会保持 HTTP 响应头的完整性，将解压工作交由调用者自己处理。

Android Ice Cream Sandwich(Android 4.0, API Level 14) 中，又增加了响应缓存，HTTP 请求将会有以下三种处理方式：

1. 完全缓存响应直接从本地存储获取，无网络情况下可以立即获得响应
2. 条件缓存响应将会通过网络服务器验证本地缓存是否过期；客户端发送请求，询问服务端本地缓存是否已过期；已过期服务端会完整下发数据；未过期则服务端返回 304 未修改状态码，这样不需要重新下发数据，节省流量并提高响应速度。
3. 未缓存响应将会从服务端下发，之后会缓存到响应缓存中。

为了保证兼容性，我们可以使用反射机制来启用响应缓存：

{% highlight java linenos %}
private void enableHttpResponseCache() {
    try {
        long httpCacheSize = 10 * 1024 * 1024; // 10 MiB
        File httpCacheDir = new File(getCacheDir(), "http");
        Class.forName("android.net.http.HttpResponseCache")
            .getMethod("install", File.class, long.class)
            .invoke(null, httpCacheDir, httpCacheSize);
    } catch (Exception httpResponseCacheNotAvailable) {
    }
}
{% endhighlight %}

使用响应缓存功能，还需要配置服务端以在响应中设置缓存 HTTP 响应头。

由于 HttpURLConnection 具有轻量性，对开发者透明的压缩机制及响应缓存的使用可带来的更快速度与省电特性，Android 团队将投入更多精力到 HttpURLConnection 并建议 Gingerbread 及以上版本中使用这个工具。

# 参考：

[Android’s HTTP Clients](http://android-developers.blogspot.com/2011/09/androids-http-clients.html "Android’s HTTP Clients")  

[HttpURLConnection](http://developer.android.com/reference/java/net/HttpURLConnection.html "HttpURLConnection")

[Connecting to the Network](http://developer.android.com/training/basics/network-ops/connecting.html "Connecting to the Network")

[关于Android的HTTP客户端的小秘密](http://blog.chengyunfeng.com/?p=196 "关于Android的HTTP客户端的小秘密")