---
layout: post
title: Camera & View
categories: [Development, Project]
---

{{ page.title }}
================
RefClass:    
	com.guo.android_extend.widget.CameraSurfaceView        
	com.guo.android_extend.widget.CameraGLSurfaceView  
	
原始的android提供了Camera 两套接口，老的接口获取图像数据时一定要设置surfaceview，使用起来非常繁琐，特别是在需要修改图像数据并且显示到屏幕上的时候会非常不方便。    

CameraSurfaceView 和 CameraGLSurfaceView 主要就是为了减少使用的复杂性。CameraSurfaceView 定义了一个统一的接口，setupCamera， setupChanged， startPreviewLater， onPreview， onBeforeRender， onAfterRender，总共6个回调时机。     

setupCamera 在SurfaceView被创建起来的时候，会被触发，需要在这个监听接口中，把相机打开并且设置参数，但不用启动预览。

setupChanged 在SurfaceView 有变化时被触发，一般情况这个SurfaceView 只用来获取数据不会触发。    

startPreviewLater 在需要启动Preview的时机会被调用，返回false则在UI创建完之后立即启动,否则不启动预览，由使用者自己控制启动时机。

onPreview 在启动Preview之后会被调用，返回相应的图像数据和时间戳，注意这里的时间戳不是底层驱动的时间戳。

onBeforeRender 在渲染图像数据传递到 GLRender 之前会触发，这里可以对图像数据再次修改，例如在上面画框之类的操作都可以。

onAfterRender 则在渲染完成之后被触发，这里可以做一些资源释放之类的操作，也可以什么都不做。

CameraSurfaceView 单独使用则渲染直接走系统流程，上面部分出函数不会触发。若配合 CameraGLSurfaceView 这个渲染显示的控件来用的的话，。CameraGLSurfaceView  这个类里使用了GLRender 这个GL 渲染类，把Camera的数据放进去就可以等比例显示，当然也可以设置不等比例的显示，支持旋转，支持输出渲染帧率。

[GLRender](https://gqjjqg.github.io/development/project/2017/11/04/GLRender.html)    



