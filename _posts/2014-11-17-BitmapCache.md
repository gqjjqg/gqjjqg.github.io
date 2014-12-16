---
layout: post
title: Bitmap cached in native memory.
categories: [Development, Project]
---

{{ page.title }}
================
之前在另外一篇 [Cache&FreshThread\[1\]](http://gqjjqg.github.io/development/project/2014/06/29/CacheThread.html)  里 有提到 Bitmap在android中是最容易引起OOM问题的对象。常用的处理方法有三个：

 1. Bitmap做java层的手动cache，需要时解码，不需要时由系统回收。 
 2. Bitmap做JNI层（C层）cache，java层需要时，直接访问JNI申请。 
 3. Bitmap直接存JNI层，采用skia的库解码，创建的也是SkaBitmap，内存完全不计入java层。
 
在[android-widget-extend](https://github.com/gqjjqg/android-widget-extend) 工程中已经实现了第一种方法，试验过第三种方法。第二种方法还没需求去做。这次乘着刚好有个项目有类似的需求，就试着把第二种方法简单实现了一下。

暂时的需求是这样的：Java层有几张非常大的Bitmap，大约每张可能6M到12M，但是java层内存吃紧，需要缓存到C层。

所以今天就试着把com.guo.android_extend.cache.BitmapCache 这个类 做了一下简单的支持。
接口有4个:

    private native int cache_init(int size);
    private native int cache_put(int handler, int hash, Bitmap bitmap);
    private native Bitmap cache_get(int handler, int hash);
    private native int cache_uninit(int handler);

BitmapCache类，逻辑功能上没有变化，新增一个destroy的接口，来释放内存。

主要是C层的实现，C层简单模拟一个LRU的算法，双向链表组织每个缓存的bitmap数据。**查询时直接O(n)匹配hash码，这里会在缓存数据个数比较大的时候存在性能问题，需要进行重写或者更换成map容器。**

--------------
在github上找了一份比较好用的C语言RBtree实现，加到了cache里，匹配速度改进到O（logN）,这份算法实现参考自Linux的底层实现，做了一下简化。

考虑到直接jni创建Bitmap在使用后废弃，系统的gc负担比较大，增加了另外一种方式，直接copy的方式，以及查询Bitmap信息的接口，以及改进创建Bitmap时，指定格式。新增接口如下：
	

    private native Bitmap cache_get(int handler, int hash, int format);
    private native int cache_search(int handler, int hash, BitmapStructure info);
    private native int cache_copy(int handler, int hash, Bitmap output);

cache部分暂时改进到这里。
