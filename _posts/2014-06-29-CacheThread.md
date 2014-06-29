---
layout: post
title: CacheThread[1]
categories: [Development, Project]
---

{{ page.title }}
================
RefClass:</br>
	com.guo.android_extend.cache.BitmapCache</br>
	
Bitmap在android中是最容易引起OOM问题的对象，但是为了显示，我们有时候又不得不去解码去使用。在过去的几个项目中，有多次都被Bitmap
的内存问题困扰，总结出的经验有三点：</br>
1.Bitmap做java层的手动cache，需要时解码，不需要时由系统回收。</br>
2.Bitmap做JNI层（C层）cache，java层需要时，直接访问JNI申请。</br>
3.Bitmap直接存JNI层，采用skia的库解码，创建的也是SkaBitmap，内存完全不计入java层</br>

这三点有各自的优点和缺点，对于1，java层的cache也会受到jvm的内存限制，节省出的内容不够多。优点是实现起来简单方便，容易使用。对于
2，java层需要bitmap时，每次都需要给定大小，再申请，使用和实现都不太方便。优点是使用完可以立即释放，尤其在bitmap大小比较固定的情况下，可以
节省相当多内存。3则是最节省内容的做法，但是兼容性有巨大的问题，实现的方法也更加复杂。</br>