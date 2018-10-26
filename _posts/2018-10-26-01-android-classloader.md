---
layout: post
keywords: Android, ClassLoader, DexClassLoader, PathClassLoader, BaseDexClassLoader, DexPathList, DexFile
description: android class load mechanism
title: "Android类加载机制"
categories: [Android]
tags: [Android, DexClassLoader, PathClassLoader, DexPathList, DexFile]
group: Android
icon: file-alt
date: 2018-10-26 11:41:00
---

# ClassLoader

Android 中使用`PathClassLoader`、`DexClassLoader`等来实现类的加载，这两个类都继承自`BaseDexClassLoader`，而`BaseDexClassLoader`继承自`ClassLoader`。

从Android官方文档中可知：[`PathClassLoader`](https://developer.android.com/reference/dalvik/system/PathClassLoader.html)用来加载系统类及已安装的应用程序的类；[`DexClassLoader`](https://developer.android.com/reference/dalvik/system/DexClassLoader.html)用来加载`jar`、`apk`、`dex`文件中的类。

`ClassLoader`使用了一种称为“双亲委托”的机制：

<!--excerpt-->

    public abstract class ClassLoader {

        private final ClassLoader parent;

        protected Class<?> loadClass(String name, boolean resolve)
            throws ClassNotFoundException
        {
                Class c = findLoadedClass(name);
                if (c == null) {
                    try {
                        if (parent != null) {
                            c = parent.loadClass(name, false);
                        } else {
                            c = findBootstrapClassOrNull(name);
                        }
                    } catch (ClassNotFoundException e) {
                    }

                    if (c == null) {
                        c = findClass(name);
                    }
                }
                return c;
        }
    }

调用`loadClass`方法来加载类时，首先使用`findLoadedClass`来获取已经被加载到内存中的类，如果类已经加载过，就会直接返回已加载的类；否则，先去调用`parent`的`loadClass`方法来加载类，`parent`无法加载时才会调用自身的`findClass`方法来加载。

# BaseDexClassLoader


    public class BaseDexClassLoader extends ClassLoader {

        private final DexPathList pathList;

        public BaseDexClassLoader(String dexPath, File optimizedDirectory,
                String libraryPath, ClassLoader parent) {
            super(parent);
            this.originalPath = dexPath;
            this.pathList =
                new DexPathList(this, dexPath, libraryPath, optimizedDirectory);
        }

        @Override
        protected Class<?> findClass(String name) throws ClassNotFoundException {
            Class clazz = pathList.findClass(name);
            if (clazz == null) {
                throw new ClassNotFoundException(name);
            }
            return clazz;
        }
    }

`BaseDexClassLoader`中`findClass`方法将类的加载委托给`DexPathList`类型的`pathList`处理，下面看下`DexPathList`

# DexPathList

`DexPathList`类关键方法如下：

    final class DexPathList {
        private static final String DEX_SUFFIX = ".dex";
        private static final String JAR_SUFFIX = ".jar";
        private static final String ZIP_SUFFIX = ".zip";
        private static final String APK_SUFFIX = ".apk";

        private final ClassLoader definingContext;
        private final Element[] dexElements;

        public DexPathList(ClassLoader definingContext, String dexPath,
                String libraryPath, File optimizedDirectory) {
            this.definingContext = definingContext;
            this.dexElements =
                makeDexElements(splitDexPath(dexPath), optimizedDirectory);
        }

        private static Element[] makeDexElements(ArrayList<File> files,
                File optimizedDirectory) {
            ArrayList<Element> elements = new ArrayList<Element>();

            for (File file : files) {
                ZipFile zip = null;
                DexFile dex = null;
                String name = file.getName();
                if (name.endsWith(DEX_SUFFIX)) {
                    // Raw dex file (not inside a zip/jar).
                    try {
                        dex = loadDexFile(file, optimizedDirectory);
                    } catch (IOException ex) {
                        System.logE("Unable to load dex file: " + file, ex);
                    }
                } else if (name.endsWith(APK_SUFFIX) || name.endsWith(JAR_SUFFIX)
                        || name.endsWith(ZIP_SUFFIX)) {
                    try {
                        zip = new ZipFile(file);
                    } catch (IOException ex) {
                        System.logE("Unable to open zip file: " + file, ex);
                    }
                    try {
                        dex = loadDexFile(file, optimizedDirectory);
                    } catch (IOException ignored) {
                    }
                } else {
                    System.logW("Unknown file type for: " + file);
                }
                if ((zip != null) || (dex != null)) {
                    elements.add(new Element(file, zip, dex));
                }
            }
            return elements.toArray(new Element[elements.size()]);
        }
    }

`DexPathList`构造函数参数`dexPath`包含了`dex`文件的列表，这些`dex`文件可以是`jar`、`zip`、`apk`格式的压缩包，也可以直接是`dex`文件。`DexPathList`使用`makeDexElements`方法遍历这些`dex`文件，使用`loadDexFile`方法加载`dex`文件：

    final class DexPathList {

        private static DexFile loadDexFile(File file, File optimizedDirectory)
                throws IOException {
            if (optimizedDirectory == null) {
                return new DexFile(file);
            } else {
                String optimizedPath = optimizedPathFor(file, optimizedDirectory);
                return DexFile.loadDex(file.getPath(), optimizedPath, 0);
            }
        }
    }

`loadDexFile`方法则使用`DexFile`的`loadDex`方法打开`dex`文件，同时会将`dex`优化为`odex`文件，返回的是`DexFile`对象，对`dex`文件的打开文件、加载类等操作都通过`DexFile`来实现。

`DexPathList.makeDexElements`将每个`dex`文件对应的`File`、`ZipFile`、`DexFile`包装成`Element`变量，这些`Element`变量存在`DexPathList`的`dexElements`数组里。

    /*package*/ static class Element {
        public final File file;
        public final ZipFile zipFile;
        public final DexFile dexFile;
        public Element(File file, ZipFile zipFile, DexFile dexFile) {
            this.file = file;
            this.zipFile = zipFile;
            this.dexFile = dexFile;
        }
        public URL findResource(String name) {
            if ((zipFile == null) || (zipFile.getEntry(name) == null)) {
                return null;
            }
            try {
                return new URL("jar:" + file.toURL() + "!/" + name);
            } catch (MalformedURLException ex) {
                throw new RuntimeException(ex);
            }
        }
    }

`DexPathList`类里加载`class`如下：

    final class DexPathList {

        public Class findClass(String name) {
            for (Element element : dexElements) {
                DexFile dex = element.dexFile;
                if (dex != null) {
                    Class clazz = dex.loadClassBinaryName(name, definingContext);
                    if (clazz != null) {
                        return clazz;
                    }
                }
            }
            return null;
        }
    }

`DexPathList`加载类时，遍历`Element`数组，使用`element.dexFile`的`loadClassBinaryName`方法来实际加载类：

    public final class DexFile {
        public Class loadClassBinaryName(String name, ClassLoader loader) {
            return defineClass(name, loader, mCookie);
        }
        private native static Class defineClass(String name, ClassLoader loader, int cookie);

    }

而`loadClassBinaryName`方法最终调用的是 native 方法`defineClass`。

# 总结

`DexClassLoader`或`PathClassLoader`加载类的过程：

    DexClassLoader.loadClass ->
        DexClassLoader.findClass ->
            BaseDexClassLoader.findClass ->
                DexPathList.findClass ->
                    DexFile.loadClassBinaryName ->
                        DexFile.defineClass

# 参考

[DexClassLoader](https://developer.android.com/reference/dalvik/system/DexClassLoader.html)

[DexClassLoader](https://android.googlesource.com/platform/libcore-snapshot/+/ics-mr1/dalvik/src/main/java/dalvik/system/DexClassLoader.java)

[PathClassLoader](https://developer.android.com/reference/dalvik/system/PathClassLoader.html)

[PathClassLoader](https://android.googlesource.com/platform/libcore-snapshot/+/ics-mr1/dalvik/src/main/java/dalvik/system/PathClassLoader.java)

[BaseDexClassLoader](https://android.googlesource.com/platform/libcore-snapshot/+/ics-mr1/dalvik/src/main/java/dalvik/system/DexClassLoader.java)

[DexPathList](https://android.googlesource.com/platform/libcore-snapshot/+/ics-mr1/dalvik/src/main/java/dalvik/system/DexPathList.java)

[DexFile](https://android.googlesource.com/platform/libcore-snapshot/+/ics-mr1/dalvik/src/main/java/dalvik/system/DexFile.java)
