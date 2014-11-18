---
layout: post
title: HListView overscroll and springback
categories: [Development,Project]
---

{{ page.title }}
================

HListView 之前就已经考虑了回弹的扩展，在CSDN论坛上看到有求回弹的水平ListView，顺便就把这个回弹的效果给实现了。更新了一下HlistView，代码量不多，核心部分在这里介绍一下，一座记录：

listView 控件本身就自带了 overScrollby的函数，用来实现回弹，但是继承的是View的函数，并且onOverScroll也没有实现。因此这里就不考虑走这条继承的路线，直接在AbsHAdapterView 这个类中实现拉过头阻尼回弹效果。

考虑到之前实现的时候并没有放footView和headView，拉过头直接就是控件的背景，这样处理起来就非常简单。只需要在原先的拖动范围0，mMaxDistanceX之外增加允许拖出的距离（mOverScrollDistance）即可。 

拖动的阻尼，可以通过距离的百分比来动态计算增量。
回滚的动画，则可以通过overScroller的springBack 来计算，这个可以参考滑动的Runnable来实现，原理是一样的。

多的也没什么特别，具体实现参考代码就行。

