---
layout: post
title: Cache&FreshThread[1]
categories: [Development, Project]
---

{{ page.title }}
================
RefClass:
	com.guo.android_extend.cache.BitmapCache
	
Bitmap在android中是最容易引起OOM问题的对象，但是为了显示，我们有时候又不得不去解码去使用。在过去的几个项目中，有多次都被Bitmap的内存问题困扰，总结出的经验有三点：
1.Bitmap做java层的手动cache，需要时解码，不需要时由系统回收。
2.Bitmap做JNI层（C层）cache，java层需要时，直接访问JNI申请。
3.Bitmap直接存JNI层，采用skia的库解码，创建的也是SkaBitmap，内存完全不计入java层

这三点有各自的优点和缺点，对于1，java层的cache也会受到jvm的内存限制，节省出的内容不够多。优点是实现起来简单方便，容易使用。对于2，java层需要bitmap时，每次都需要给定大小，再申请，使用和实现都不太方便。优点是使用完可以立即释放，尤其在bitmap大小比较固定的情况下，可以节省相当多内存。3则是最节省内容的做法，但是兼容性有巨大的问题，实现的方法也更加复杂。

1. 实现需要依靠LinkedHashMap，以及java的“软链接”。正常情况下一个LRU的LinkedHaspMap是较为适合做cache的，这样加入的顺序也会成为一个需要考虑的因素。基本的实现如下：    

```java
public class LRULinkedHashMap<K, V> extends LinkedHashMap<K, V> {
	/**
	 * @Version InitVersion. 10000L
	 */
	private static final long serialVersionUID = 10000L;

	private int mMaxSize;

	public LRULinkedHashMap(int initialCapacity, float loadFactor,
			boolean accessOrder) {
		super(initialCapacity, loadFactor, accessOrder);
		// TODO Auto-generated constructor stub
		mMaxSize = initialCapacity;
	}

	@Override
	protected boolean removeEldestEntry(Entry<K, V> eldest) {
		// TODO Auto-generated method stub
		return size() > mMaxSize;
	}

}
```    

这样就可以作为cache的容器来使用。我们再给它包装一下，因为K并不一定就是整形ID，也可能是String的路径，也可能是其他的类型。    

```java
package com.guo.android_extend.cache;

import java.lang.ref.SoftReference;
import java.util.HashMap;
import com.guo.android_extend.java.LRULinkedHashMap;

import android.graphics.Bitmap;

public class BitmapCache<T> {

	private HashMap<T, SoftReference<Bitmap>> mCacheMap;

	private int CACHE_SIZE;

	/**
	 * default cache size is 12
	 */
	public BitmapCache() {
		this(12);
	}

	public BitmapCache(int CacheSize) {
		CACHE_SIZE = CacheSize;
		mCacheMap = new LRULinkedHashMap<T, SoftReference<Bitmap>>(CACHE_SIZE,
				0.75F, true);
	}

	public synchronized void putBitmap(T id, SoftReference<Bitmap> bm) {
		mCacheMap.put(id, bm);
	}

	public synchronized SoftReference<Bitmap> getBitmap(T id) {
		return mCacheMap.get(id);
	}

}
```    

包装之后的Cache使用时需要指定用什么作为Key来标志Bitmap。这部分是比较常用的功能，因此已经放入项目中，可以随时取用。    

2. 实现需要JNI支持，首先初始化时在JNI层指定好cache的大小，因为JNI的内存就是系统的实际内容，也不是无穷无尽的。具体的做法就因需求而定了。基本思路和Java的代码思路类似。因为项目中没有加入，这里就不细说了。

3. 实现需要android的系统的Skia头文件，以及.a来编译通过。我们考察过了2.3.3，4.1.2 以及3.0的平台，发现Skia头文件更新非常多，因此，可能每个不同的平台都要一个对应的so。加载时还要判断平台版本来加载对应的so。做起来非常复杂，因此，这里就不考虑了。实际上，我们已经可以做到这种实现的方式。得益于android开源的代码，我们可以非常深入的查阅java 层Bitmap对于Skia的Bitmap的使用，参考之后基本就不会有问题。唯一值得注意的是编译连接时，需要把不同平台的.a拿出来编译。这就更增加了兼容的难度。谁知道以后会不会有5.0，6.0，或者不使用Skia图形引擎的情况出现。

cache 这部分的内容就到此为止了。
接下去，要介绍的是提升getView的性能，避免滑动的卡顿。    

[Cache&FreshThread[1]](http://gqjjqg.github.io/development/project/2014/06/29/CacheThread2.html)    

