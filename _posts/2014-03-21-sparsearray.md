---
layout: post
keywords: android, SparseArray
description: SparseArray分析
title: "SparseArray分析"
categories: [Android]
tags: [android, SparseArray]
group: Android
icon: file-alt
---
{% include JB/setup %}

SparseArray（稀疏数组）是android提供的建议替换HashMap&lt;Integer, E&gt;的用来存储整型、对象键值对的类。它比HashMap&lt;Integer, E&gt;要高效轻量。其内部使用了两个数组来分别缓存key和value，查找使用折半通过key来查找。

SparseArray提供以下方法：

###构造

SparseArray提供了两个构造函数：

    public SparseArray();//默认初始化容器大小为10

    public SparseArray(int initialCapacity);

###增

    public void put(int key, E value);//增加或替换

    public void append(int key, E value);//同put，对key大于所有已缓存key情况进行了优化

###删

    public void delete(int key);

    public void remove(int key);//同delete

    public void removeAt(int index);

    public void clear();

###改

    public void setValueAt(int index, E value);//设置序号index处value

###查

    public E get(int key);//没有则返回null

    public E get(int key, E valueIfKeyNotFound);//没有返回valueIfKeyNotFound

    public int keyAt(int index);

    public E valueAt(int index);

    public int indexOfKey(int key);

    public int indexOfValue(E value);

<!--excerpt-->

###高效性

SparseArray在进行删除单一键值对操作时，并不是直接移除对应键值对，而是将对应的value设置为删除标记对象DELETED，并设置标记mGarbage为true，在调用gc()时才会真的进行删除操作：进行数组拷贝并设置mSize减一。这样，如果再次设置了同样key的键值对，就不需要重新调整内部数组，直接设置value。gc的操作以下情况调用：1.put或append时mGarbage标记为true且内部键值对数组已满；2.进行查询操作；

SparseArray的高效性还体现在其内部使用了数组来存储keys和values，其查找使用key来折半查找，没有找到则返回应该插入的位置，这样查找和插入都很高效。

###源码分析

    /**
     * SparseArray提供了整型到Object对象的映射，其索引不连续，这点和普通
     * 的对象数组不同。SparseArray比HashMap<Integer, E>更高效
     */
    public class SparseArray<E> implements Cloneable {
        private static final Object DELETED = new Object();//标记对象已删除
        private boolean mGarbage = false;//标记有对象已被删除，可以启动回收

        private int[] mKeys;//所有key的集合
        private Object[] mValues;//所有values的集合
        private int mSize;//键值对的个数

        public SparseArray() {//默认初始化容器大小为10
            this(10);
        }

        public SparseArray(int initialCapacity) {
            initialCapacity = ArrayUtils.idealIntArraySize(initialCapacity);
            mKeys = new int[initialCapacity];
            mValues = new Object[initialCapacity];
            mSize = 0;
        }

        @Override
        @SuppressWarnings("unchecked")
        public SparseArray<E> clone() {//clone方法
            SparseArray<E> clone = null;
            try {
                clone = (SparseArray<E>) super.clone();
                clone.mKeys = mKeys.clone();
                clone.mValues = mValues.clone();
            } catch (CloneNotSupportedException cnse) {
                /* ignore */
            }
            return clone;
        }

        /**
         * 根据key查找Object对象并返回，没有则返回null
         */
        public E get(int key) {
            return get(key, null);
        }

        /**
         * 根据key查找Object对象并返回，没有则返回valueIfKeyNotFound
         */
        @SuppressWarnings("unchecked")
        public E get(int key, E valueIfKeyNotFound) {
            int i = binarySearch(mKeys, 0, mSize, key);
    
            if (i < 0 || mValues[i] == DELETED) {
                return valueIfKeyNotFound;
            } else {
                return (E) mValues[i];
            }
        }

        /**
         * 根据key删除对应键值对，实际没有删除只是标记到已删除对象DELETED，下次调用gc()时回收
         */
        public void delete(int key) {
            int i = binarySearch(mKeys, 0, mSize, key);
    
            if (i >= 0) {
                if (mValues[i] != DELETED) {
                    mValues[i] = DELETED;
                    mGarbage = true;
                }
            }
        }

        /**
         * 同delete(int)
         */
        public void remove(int key) {
            delete(key);
        }

        /**
         * 删除制定index位置的键值对，实际也是做标记，gc时删除
         */
        public void removeAt(int index) {
            if (mValues[index] != DELETED) {
                mValues[index] = DELETED;
                mGarbage = true;
            }
        }

        /**
         * 垃圾回收，
         */
        private void gc() {    
            int n = mSize;
            int o = 0;
            int[] keys = mKeys;
            Object[] values = mValues;

            for (int i = 0; i < n; i++) {
                Object val = values[i];

                if (val != DELETED) {
                    if (i != o) {
                        keys[o] = keys[i];
                        values[o] = val;
                        values[i] = null;
                    }    
                    o++;
                }
            }
            mGarbage = false;
            mSize = o;
        }

        /**
         * 添加键值对，会覆盖同key的键值对
         */
        public void put(int key, E value) {
            int i = binarySearch(mKeys, 0, mSize, key);

            if (i >= 0) {
                mValues[i] = value;
            } else {
                i = ~i;

                if (i < mSize && mValues[i] == DELETED) {
                    mKeys[i] = key;
                    mValues[i] = value;
                    return;
                }

                if (mGarbage && mSize >= mKeys.length) {
                    gc();

                    // Search again because indices may have changed.
                    i = ~binarySearch(mKeys, 0, mSize, key);
                }

                if (mSize >= mKeys.length) {
                    int n = ArrayUtils.idealIntArraySize(mSize + 1);

                    int[] nkeys = new int[n];
                    Object[] nvalues = new Object[n];

                    System.arraycopy(mKeys, 0, nkeys, 0, mKeys.length);
                    System.arraycopy(mValues, 0, nvalues, 0, mValues.length);

                    mKeys = nkeys;
                    mValues = nvalues;
                }

                if (mSize - i != 0) {
                    System.arraycopy(mKeys, i, mKeys, i + 1, mSize - i);
                    System.arraycopy(mValues, i, mValues, i + 1, mSize - i);
                }

                mKeys[i] = key;
                mValues[i] = value;
                mSize++;
            }
        }

        /**
         * 返回当前存储的键值对数量
         */
        public int size() {
            if (mGarbage) {
                gc();
            }
            return mSize;
        }

        /**
         * 返回序号index位置上的key
         */
        public int keyAt(int index) {
            if (mGarbage) {
                gc();
            }
            return mKeys[index];
        }

        /**
         * 返回序号index位置上的value
         */
        public E valueAt(int index) {
            if (mGarbage) {
                gc();
            }
            return (E) mValues[index];
        }

        /**
         * 直接设置序号index位置的value
         */
        public void setValueAt(int index, E value) {
            if (mGarbage) {
                gc();
            }
            mValues[index] = value;
        }

        /**
         * 返回给定key所在序号，不存在则返回应该插入的位置序号取反；
         * 查找前先gc
         */
        public int indexOfKey(int key) {
            if (mGarbage) {
                gc();
            }
            return binarySearch(mKeys, 0, mSize, key);
        }

        /**
         * 遍历value数组，返回给定value所在的序号，不再列表中返回-1
         * 遍历前先gc
         * 注意：遍历是线性的，而且多个key可能映射到同一个value，这时只会返回第一个序号
         */
        public int indexOfValue(E value) {
            if (mGarbage) {
                gc();
            }
            for (int i = 0; i < mSize; i++)
                if (mValues[i] == value)
                    return i;

            return -1;
        }
    
        /**
         * 清除SparseArray中所有键值对
         */
        public void clear() {
            int n = mSize;
            Object[] values = mValues;
            for (int i = 0; i < n; i++) {
                values[i] = null;
            }
            mSize = 0;
            mGarbage = false;
        }

        /**
         * 添加键值对到数组中，这里对key大于所有已存在的key的情况进行了优化，
         * 其它情况直接调用put(int, E)
         */
        public void append(int key, E value) {
            if (mSize != 0 && key <= mKeys[mSize - 1]) {
                put(key, value);
                return;
            }

            if (mGarbage && mSize >= mKeys.length) {
                gc();
            }

            int pos = mSize;
            if (pos >= mKeys.length) {
                int n = ArrayUtils.idealIntArraySize(pos + 1);
                int[] nkeys = new int[n];
                Object[] nvalues = new Object[n];

                System.arraycopy(mKeys, 0, nkeys, 0, mKeys.length);
                System.arraycopy(mValues, 0, nvalues, 0, mValues.length);

                mKeys = nkeys;
                mValues = nvalues;
            }

            mKeys[pos] = key;
            mValues[pos] = value;
            mSize = pos + 1;
        }

        /**
         * 使用二分查找法找到key在数组a中的位置，若不存在，返回指定key应该插入的位置的序号取反，
         * 即序号区负再减一，这样便于插入
         */
        private static int binarySearch(int[] a, int start, int len, int key) {
            //high和low初始值超出给定查找范围，为了能定位到start位置和start+len位置
            int high = start + len, low = start - 1, guess;
            while (high - low > 1) {
                guess = (high + low) / 2;
                if (a[guess] < key)
                    low = guess;
                else
                    high = guess;
            }

            if (high == start + len)//key大于数组中所有值，high超出数组给定范围
                return ~(start + len);
            else if (a[high] == key)//查找到key，返回序号
                return high;
            else//key不大于数组中最大值
                return ~high;
        }
    }

###SparseBooleanArray、SparseIntArray

SparseBooleanArray是设计用来替换HashMap&lt;Integer, Boolean&gt;，而SparseIntArray是用来替换HashMap&lt;Integer, Integet&gt;，他们内部处理和SparseArray基本一致，只有一点不同：它们的单个键值对的删除操作是直接进行数组拷贝，而SparseArray是先标记为DELETED，后面需要时再删除。

***
#参考:

[Android编程之SparseArray&lt;E&gt;详解](http://blog.csdn.net/xyz_fly/article/details/7931943)

[Android应用性能优化之使用SparseArray替代HashMap](http://liuzhichao.com/p/832.html)