---
layout: post
keywords: Java
description: introduct the try-with-resources Statement of java 7
title: "Java 之 try-with-resources"
categories: [Java]
tags: [Java, try, AutoCloseable]
group: Java
icon: file-alt
date: 2018-03-18 15:42:00
---

`Java`中打开的某些资源需要我们在使用完成后主动关闭，譬如输入输出流：


	final InputStream in = con.getInputStream();
	final FileOutputStream os = new FileOutputStream(file);
	
	final byte[] buf = new byte[1024 * 8];
	
	int len;
	
	while (-1 != (len = in.read(buf))) {
	    os.write(buf, 0, len);
	}
	
	os.close();
	in.close();

但是我们有时候忘了主动关闭，或者逻辑有疏漏导致无法关闭，譬如以上代码在`os.close()`调用时抛出异常，就会导致`in.close()`无法执行到。

# `try-with-resources`语句

在`Java7`中新增了`try-with-resources`语句，提供了资源关闭的新方法。

<!--excerpt-->

首先来介绍下`资源(resource)`的含义；一个资源即一个在程序完成后必须被关闭的对象(object)。

而`try-with-resources`语句就是声明了一个或多个资源的`try`语句，声明资源的语句放到`try`关键字后的`圆括号()`内，多个资源的声明以`分号;`分隔。

譬如下面就是声明了两个资源的`try-with-resources`语句：

	try (
	        final InputStream in = con.getInputStream();
	        final FileOutputStream os = new FileOutputStream(patchFile)
	) {
	
	    ...
	}

那么什么样的对象才能作为`资源(resource)`声明在`try-with-resources`语句中呢？

# `java.lang.AutoCloseable`

在`Java7`中新增了接口[`java.lang.AutoCloseable`](https://docs.oracle.com/javase/7/docs/api/java/lang/AutoCloseable.html)：

	package java.lang;

	/**
	 * @since 1.7
	 */
	public interface AutoCloseable {
	
	    void close() throws Exception;
	}

所有实现了(implements)[`java.lang.AutoCloseable`](https://docs.oracle.com/javase/7/docs/api/java/lang/AutoCloseable.html)接口的类的实例，都可以作为可关闭的`资源(resource)`在`try-with-resources`语句中声明。这也包括所有实现了[`java.io.Closeable`](https://docs.oracle.com/javase/7/docs/api/java/io/Closeable.html)接口的对象，因为在`Java7`中[`java.io.Closeable`](https://docs.oracle.com/javase/7/docs/api/java/io/Closeable.html)接口继承了[`java.lang.AutoCloseable`](https://docs.oracle.com/javase/7/docs/api/java/lang/AutoCloseable.html)接口：

	package java.io;
	
	import java.io.IOException;
	
	/**
	 * @since 1.5
	 */
	public interface Closeable extends AutoCloseable {
	
	    public void close() throws IOException;
	}

# 关闭资源

在`Java7`以前，我们需要使用`finally`语句块来保证`try`语句正常执行完成或异常中止时，资源都能够正确的关闭。

如下代码从文件中读取一行文本，使用了`BufferedReader`对象的实例来从文件中读取数据，这个对象在程序执行完后必须关闭：

	static String readFirstLineFromFileWithFinallyBlock(String path)
	                                                     throws IOException {
	    BufferedReader br = new BufferedReader(new FileReader(path));
	    try {
	        return br.readLine();
	    } finally {
	        if (br != null) br.close();
	    }
	}

而使用`try-with-resources`语句，简化为如下：

	static String readFirstLineFromFile(String path) throws IOException {
	    try (BufferedReader br =
	                   new BufferedReader(new FileReader(path))) {
	        return br.readLine();
	    }
	}

在`Java7`中，`BufferedReader`实现了`java.lang.AutoCloseable`接口。将`BufferedReader`实例的声明放到`try-with-resources`语句里，当程序正常执行完或异常中止退出执行，`BufferedReader`实例都会被自动关闭。

# 多个资源关闭顺序

`try-with-resources`语句可以声明一个或多个资源。下面的例子从`zip`文件中获取打包的文件的名称并创建一个文本文件保存压缩包中的文件名称：

	public static void writeToFileZipFileContents(String zipFileName,
	                                           String outputFileName)
	                                           throws java.io.IOException {
	
	    java.nio.charset.Charset charset = java.nio.charset.StandardCharsets.US_ASCII;
	    java.nio.file.Path outputFilePath = java.nio.file.Paths.get(outputFileName);
	
	    try (
	        java.util.zip.ZipFile zf = new java.util.zip.ZipFile(zipFileName);
	        java.io.BufferedWriter writer = java.nio.file.Files.newBufferedWriter(outputFilePath, charset)
	    ) {
	        for (java.util.Enumeration entries = zf.entries(); entries.hasMoreElements();) {
	            String newLine = System.getProperty("line.separator");
	            String zipEntryName = ((java.util.zip.ZipEntry)entries.nextElement()).getName() + newLine;
	            writer.write(zipEntryName, 0, zipEntryName.length());
	        }
	    }
	}

这个例子中的`try-with-resources`语句中顺序声明了`ZipFile`、`BufferedWriter`的实例，当程序正常或异常执行完成后，资源按照其声明顺序的逆序被依次关闭，即先关闭`BufferedWriter`的实例`ZipFile`，再关闭`ZipFile`实例。

# Suppressed Exceptions

上面读取文件一行文本的代码，如果`readLine`和资源关闭的`close`方法都抛出了异常，两种方式抛出的异常不同。

## `finally`语句块

`readFirstLineFromFileWithFinallyBlock`函数将抛出`finally`语句块内抛出的异常，`try`语句块抛出的异常则被`抑制(suppressed)`了。

## `try-with-resources`语句

相反，如果`readFirstLineFromFile`函数在`try`语句块与`try-with-resources`语句中分别抛出了异常，函数会抛出`try`语句块抛出的异常，而`try-with-resources`语句中抛出的异常则被`抑制(suppressed)`了。

其中，`try`语句块中可以抛出一个异常，而`try-with-resources`语句中可以抛出一个或多个异常。

譬如上面读取`zip`压缩包中文件名称的`writeToFileZipFileContents`函数中，`try`语句块中可以抛出一个异常，而`try-with-resources`语句中可以抛出最多两个异常，即`ZipFile`与`BufferedWriter`实例分别关闭抛出的异常。

当`try-with-resources`语句中抛出的异常被`try`语句块中抛出的异常`抑制(suppressed)`时，可以使用`try`语句块中抛出的异常对象调用`Throwable.getSuppressed`方法来获取被`抑制(suppressed)`的异常：

	package java.lang;
	import  java.io.*;
	import  java.util.*;
	
	public class Throwable implements Serializable {
	
	    /**
	     * Returns an array containing all of the exceptions that were
	     * suppressed, typically by the {@code try}-with-resources
	     * statement, in order to deliver this exception.
	     *
	     * If no exceptions were suppressed or {@linkplain
	     * #Throwable(String, Throwable, boolean, boolean) suppression is
	     * disabled}, an empty array is returned.  This method is
	     * thread-safe.  Writes to the returned array do not affect future
	     * calls to this method.
	     *
	     * @return an array containing all of the exceptions that were
	     *         suppressed to deliver this exception.
	     * @since 1.7
	     */
	    public final synchronized Throwable[] getSuppressed() {
	        ...
	    }
	}

# 参考阅读

[The try-with-resources Statement
](https://docs.oracle.com/javase/tutorial/essential/exceptions/tryResourceClose.html)