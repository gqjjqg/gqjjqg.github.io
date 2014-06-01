---
layout: post
title: 水平滑动ListView
categories: [Development]
---

{{ page.title }}
================

在android-widget-extend 的开源项目中 做了一个水平滑动的ListView。 前段时间花了很多时间做UI特效，为项目做的一个基础特效在Dev分支上更新了一下。 特效大概是这样的，左右滑动所有View 大小一致，停止时，中间的View放大，向上滑动时删除中间的View。 目前还有一些BUG:

 1. 处于未显示状态的View会在删除时闪现。
 2. 中间飞出的View会由于ListView的刷新，出现突变，从放大突然缩小到正常消失。

功能上可以正常使用。

