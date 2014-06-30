---
layout: post
title: CacheThread[1]
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

1的实现需要依靠LinkedHashMap，以及java的“软链接”。正常情况下一个LRU的LinkedHaspMap是较为适合做cache的，这样加入的顺序也会成为一个需要考虑的因素。基本的实现如下：

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

