---
layout: post
title: Start create your own robot
categories: [Development, Blog]
---

{{ page.title }}
================
最近买了块开发板，开始尝试自己搭建一个智能的移动平台。
开发板：CubieBoard 4(CC-A80)
配件：一个串口转microUSB.以及各种线
移动平台：DFRobot的一个学习套间，海盗船。

开发板没有什么好介绍的，功能强大，接口齐全，目前唯一不足的是MIPI没有接出，没法接高清摄像头。

移动平台，是以DFRobot的 Romeo V1.0 ble 主控板，用的芯片还是Arduino UNO 。此移动平台资料也比较齐全，经过一步步马达调试，舵机调试，超声波调试，蓝牙调试，现在也已经可以使用了。 舵机因为安装位置的关系，要和马达的电源线路要远离，否则会受到干扰。 

移动平台搭建完之后，烧写的代码已经支持串口输入，蓝牙输入。接下去就要把CC-A80的串口和主控板用串口连接，通过A80作为主要的工作平台，连接Arduino，A80作为MINI PC，负责大数据量，高负荷的运算，比如外接摄像头，传输数据等。

A80 的ttyS0是作为 boot的log输出，因此不适合拿来通信，ttyS1是已经被无线模块占用，ttyS2和蓝牙模块共用，也不适合，ttyS3 对应串口4 是没有被使用，并且有引脚接出。

接好线路，接下去就修改固件，打开串口4。
下载源码，找到路径：
lichee/tools/pack/chips/sun9iw1p1/configs/cubieboard4/sys_config.fex 
启用uart4之后，按照编译文档指示，重新编译内核。

接下去测试一下串口4是否正常工作，剩下的就是软件应用的问题了。


最后的调试图：
<image src="http://gqjjqg.github.io/images/IMG_20150529_172617.jpg" width="640" height="480" />

