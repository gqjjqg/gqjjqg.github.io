---
layout: post
title: RotateDectection[2]
categories: [Development, Project]
---

{{ page.title }}
================

RefClass:    
	com.guo.android_extend.widget.ExtImageButton    
	com.guo.android_extend.widget.ExtImageView    
	com.guo.android_extend.widget.ExtRelativeLayout    
	com.guo.android_extend.RotateRunable    
	
[RotateDectection[1]](http://gqjjqg.github.io/development/project/2014/06/24/RotateDectection.html) 介绍了旋转检测和注册接口。这里介绍一下旋转动画的实现。    
考虑到4个方向旋转，View的旋转可以分成 顺时针旋转90/逆时针旋转90/旋转180三种情况。这样旋转动画可以简单实现：
```java
@Override
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
}
```
    
degree为当前旋转角度，offset为相对之前旋转的度数，flag为顺时针或者逆时针的标记。均会由
com.guo.android_extend.CustomOrientationDetector计算传入。offset在顺时针时为正，逆时针时为负。
这样旋转的动画就完成了，在最后用handler去post一个动画的runanble即可。这个runnable也在com.guo.android_extend下。
当然不同的View可能当前的角度各不相同（这种只有在动态增加的情况下可能产生），因此必须要实现一个getCurrentOrientationDegree的方法，
这个方法是告诉CustomOrientationDetector当前这个注册监听的对象当前的角度是多少，用来计算旋转offset和degree之用。    
这样计算角度和旋转的部分都集中统一于CustomOrientationDetector。    
    
旋转显示的核心则是旋转画布，在View的onDraw中重写即可：    
```java
@Override
protected void onDraw(Canvas canvas) {
	// TODO Auto-generated method stub
	canvas.save();
	canvas.rotate(-mCurDegree, this.getWidth() / 2f, this.getHeight() / 2f);
	super.onDraw(canvas);
	canvas.restore();
}
```
值得注意的是布局需要调用一个setWillNotDraw来让OnDraw生效。
布局的画布略有不同：    
```java
@Override
protected void onDraw(Canvas canvas) {
	// TODO Auto-generated method stub
	super.onDraw(canvas);
	canvas.rotate(-mCurDegree, this.getWidth() / 2f, this.getHeight() / 2f);
}
```
在Android 3.0之后android支持布局直接rotate，3.0之前只能把画布旋转之后，同时还要把点击的点重写映射，这样才能和布局的点击位置一致。这部分的代码可能随着时间推移，2.3.3的减少会逐渐废弃，目前的处理方法是先旋转MotionEvent中的点，再重新dispatch。
```java
@Override
public boolean dispatchTouchEvent(MotionEvent ev) {
	PointF newPoint = rotatePoint(new PointF(ev.getX(), ev.getY()),
			new PointF(this.getWidth() / 2F, this.getHeight() / 2F),
			-mCurDegree);
	MotionEvent newEvent = MotionEvent.obtain(ev.getDownTime(),
			ev.getEventTime(), ev.getAction(), newPoint.x, newPoint.y,
			ev.getPressure(), ev.getSize(), ev.getMetaState(),
			ev.getXPrecision(), ev.getYPrecision(), ev.getDeviceId(),
			ev.getEdgeFlags());
	// TODO Auto-generated method stub
	return super.dispatchTouchEvent(newEvent);
}
/**
 * @param A
 * @param B center point
 * @param degree
 * @return
 */
private PointF rotatePoint(PointF A, PointF B, float degree) {
	float radian = (float) Math.toRadians(degree);
	float cos = (float) Math.cos(radian);
	float sin = (float) Math.sin(radian);
	float x = (float) ((A.x - B.x)* cos +(A.y - B.y) * sin + B.x);  
	float y = (float) (-(A.x - B.x)* sin + (A.y - B.y) * cos + B.y);  
	return new PointF(x, y);
}
```

这部分控件，可以用来制作环形菜单，目前sample中没有包含，考虑以后增加。
