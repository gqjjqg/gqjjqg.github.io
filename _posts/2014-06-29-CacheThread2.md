---
layout: post
title: Cache&FreshThread[2]
categories: [Development, Project]
---

{{ page.title }}
================
RefClass:
  com.guo.android_extend.cache.BitmapMonitor
  com.guo.android_extend.cache.BitmapMonitorThread
  
接着介绍getView的性能提升，google之前就有介绍过，性能不够是因为findViewByID，因此建议大家使用Holder的形式，设置TAG，并且复用View。经过自己的试验和网上的公布的测试结果，的确是行之有效的手段。
