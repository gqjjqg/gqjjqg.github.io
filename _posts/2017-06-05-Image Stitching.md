---
layout: post
title: Image stitching
categories: [Algorithm , Blog]
---

{{ page.title }}
================
OpenCV 里有image stitch的算法，参考了一下Image Alignment and Stitching: A Tutorial 

基本的流程：
特征点搜索 SURF
特征点匹配
计算H变换矩阵
估算相机Focal
柱面投影
计算旋转矩阵
统一坐标系
波形矫正
图像warp
图像融合

通过简化流程计算如下步骤
特征点搜索 SURF
特征点匹配
计算H变换矩阵
统一坐标系
图像warp

<img src="http://gqjjqg.github.io/images/result_stitch1.jpg" width="640" height="320" />

可以明显看到越往右边拉伸形变越严重。
参考opencv 往里面增加柱面投影来修正这个问题。
在实验中，发现自己抄的SURF特征点检测数量远少于opencv，H矩阵计算结果也有很大偏差。
正向投影计算范围已经正确
逆向投影计算却完全不对，还需要继续检查问题所在。
