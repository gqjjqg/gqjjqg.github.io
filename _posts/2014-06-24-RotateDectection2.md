---
layout: post
title: RotateDectection[2]
categories: [Development, Project]
---

{{ page.title }}
================

RotateDectection[1] 介绍了旋转检测和注册接口。这里介绍一下旋转动画的实现。</br>
考虑到4个方向旋转，View的旋转可以分成 顺时针旋转90/逆时针旋转90/旋转180三种情况。这样旋转动画可以简单实现：
<code>@Override
	public boolean OnOrientationChanged(int degree, int offset, int flag) {
		// TODO Auto-generated method stub
		if (!this.isShown()) {
			Log.i(TAG, "Not Shown!");
			return false;
		}
		
		Animation animation = new RotateAnimation (offset, 0,
				Animation.RELATIVE_TO_SELF, 0.5f, 
				Animation.RELATIVE_TO_SELF, 0.5f);
		animation.setDuration(ANIMATION_TIME);
		animation.setFillAfter(true);
		mHandler.post(new RotateRunable(animation, this, degree));
		
		mCurDegree = degree;
		return true;
	}</code> </br>
degree为当前旋转角度，offset为相对之前旋转的度数，flag为顺时针或者逆时针的标记。均会由
com.guo.android_extend.CustomOrientationDetector计算传入。offset在顺时针时为正，逆时针时为负。
这样旋转的动画就完成了，在最后用handler去post一个动画的runanble即可。这个runnable也在com.guo.android_extend下。
当然不同的View可能当前的角度各不相同（这种只有在动态增加的情况下可能产生），因此必须要实现一个getCurrentOrientationDegree的方法，
这个方法是告诉CustomOrientationDetector当前这个注册监听的对象当前的角度是多少，用来计算旋转offset和degree之用。</br>
这样计算角度和旋转的部分都集中统一于CustomOrientationDetector。

