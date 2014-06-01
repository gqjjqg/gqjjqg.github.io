---
layout: post
title: 缓存和延迟刷新View
categories: [Development]
---

{{ page.title }}
================

        最近在看以前做的项目，有很多东西可以复用，今天来记录一下缓存bitmap和延迟刷新View。
    以往的做法是 建立一个LRU的Bitmap缓存列表，用弱引用来关联Bitmap。每次需要Bitmap时向缓存列表申请。申请不到则向子线程请求下载并解码。后续的工作就是通知UI线程，Bitmap已经准备好了，可以显示了。
        整个过程并不复杂，实现起来也比较容易。在通知UI线程刷新的这个过程中，需要考虑的东西反而是在项目中最耗费精力的。因为有时候在ListView或者一些View会复用的情况下，开发人员是很难定位取得这些View来做独立的刷新的。
        目前我们针对ListView复用的情况，制作了一个组件，可以较为轻松的管理缓存和延迟刷新View的问题。会在未来几天，从商业项目中剥离出来放入 android-widget-extend 项目中。

