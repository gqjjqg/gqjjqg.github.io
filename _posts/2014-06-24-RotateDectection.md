---
layout: post
title: RotateDectection
categories: [Development, Project]
---

{{ page.title }}
================

旋转检测是在做camera相关的项目时提取的。为了固定activity方向，但是又支持横竖屏，通过注册OrientationEventListener来获得当前设备旋转角度。通过计算旋转角度，然后旋转布局上的各个按钮或者部分控件，达到适配屏幕旋转的目的。这种方式不同于真正的区分横竖屏的独立布局，布局设计上更加简单也更容易控制，用户体验上也比较好。</br>
</br>
在项目中提取出来的主要类是 com.guo.android_extend.CustomOrientationDetector</br>
这个类实现了OrientationEventListener接口，直接被注册到android系统。</br>
通常情况下考虑4个方向的角度，区分成4个区间：</br>
   *  divider:  </br>
   *  	1. (45, 135]	return 90</br>
   * 	2. (135, 225]	return 180</br>
   * 	3. (225, 315]	return 270</br>
   * 	4. (315, 45]	return 0</br>

通过当前的旋转角度对比onOrientationChanged方法获得的角度计算是否有超过45度的旋转。 
OnOrientationListener 是CustomOrientationDetector里的接口，主要负责在角度旋转时通知注册过的控件旋转。</br>
</br>
forceOrientationChanged方法则是旋转的主要处理方法，在这个里面通过判断 OnOrientationChanged 这个方法的返回值来维护当前注册的列表
通常情况下，是一些View旋转，只需要判断View是否显示即可决定是否要保留在注册列表中。
而不是View的则需要使用者自己根据需求来返回。
比如在ListView中，在getView时注册到监听列表（可以重复注册），在某些View不可见时，旋转时会移除。</br>

CustomOrientationDetector 这个类主要的工作就是 处理了4个方向上旋转的检测，并且维护一个自己的监听列表，
sample工程里有提供相关的测试示例代码，可以自行参考。</br>
