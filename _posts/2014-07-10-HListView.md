---
layout: post
title: Horizontal ListView
categories: [Development, Project]
---

{{ page.title }}
================
RefClass:</br>
com.guo.android_extend.widget.AbsHAdapterView</br>
com.guo.android_extend.widget.HListView

横向的ListView一直是比较常用的控件，一般情况下，能达到横向ListView的效果有4种方法：

1. 强制整体布局为横屏，直接布置ListView，竖屏时ListView自然就是水平滑动了。这种方法适合布局简单的应用，并且有比较多的布局限制。因此并不建议采用。
2. 不强制整体布局的情况下，可以供直接使用的横向滑动控件是gridView。计算好item的高度，动态计算item的个数，来设置gridView的Column数量。这样可以得到比较满意的效果，这个唯一不好的地方是布局时需要代码动态计算，使用起来不够方便。
3. 采用LinearLayout结合HorizontalScrollView，布局非常方便简单，也不需要计算布局，但是每次数据更新时，需要开发者自己addView，removeView，虽然非常灵活，但是在一些子view数量较多时，会耗费较多的资源，难以快速开发。
4. 采用旋转布局的方式，强制旋转ListView的父布局，自然ListView也跟着旋转成水平滑动了。这种的限制是父布局必须要是正方形的，否则旋转之后，ListView就会被盖住一部分。

综合4种方法，比较常用的还是2，gridView的灵活性，还是值得肯定的。2的计算量和使用起来复杂度也算可以接受。

但是为了更好更快速的开发，一个原生的横向的ListView还是必要的。因为之前做过了横向滑动的控件，再参考android的ListView的实现，实现了一个HListView，支持横向的ListView，支持部分常用的功能和ListView使用一样。里面实现不算太难，复杂的主要还是内部的各种事件，并且交织处理逻辑。看过android的ListView实现之后，相信会有一个大概的印象。

大概包含了四个部分：</br>
1. 布局的计算
2. 子View的位置布局复用和滑动处理
3. click和longclick处理
4. listener的处理

布局主要需要支持warp_content，match_parent,以及具体DP。 这部分的核心在于，设置Adapter之后调用getCount,measure child的width
加上divider的宽度，计算整体的宽度和滑动的宽度。

子View的位置布局和复用和滑动略微复杂，原理上例如： 

A.---- - | 0 1 2 3 4 | ----  初始布局</br>
B.---- 0 | 1 2 3 4 - | ----  偏移布局</br>
C.---- 0 | 1 2 3 4 5 | ----  填充不足的元素</br>
D.---- - | 1 2 3 4 5 | ----  回收看不到的元素</br>

竖线之内是当前显示的几个子View。从A到B，需要把布局整体向左偏移deltaX。View0的布局right值是本身宽度，现在要减掉deltaX，变成了负数，View4的right则小于了右边的竖线。达到了B的状态之后，把View5通过Adapter.getView获取并加入布局，直到新加入的View小于右边竖线。到达C状态之后，从最右边的元素开始判断是否出了左边竖线，出了就回收掉。左右滑动都是同一原理，更详细的可以参考代码。

点击和长按事件的支持，主要还是依靠AdapterView的performanClick来传递点击。唯一值得注意的是在touchdown时post一个runnable计时在在touchup时检查，这个时间标记点是否已经过去，过去了，则不是一个click，没过去则是一个click。这个建议参考手势检测类里的实现。

listener的处理，主要剩下滚动的状态和位置监听。实现滑动和滚动布局之后，相信这部分就不会是难事。
还是一句话，建议参考sample 代码和源码。





