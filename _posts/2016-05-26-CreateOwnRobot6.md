---
layout: post
title: Start create your own robot(6)
categories: [Development, Blog]
---

{{ page.title }}
================
基本上优必选的机器人能做的也就是动作的控制，LED和喇叭之类的基础功能。要想做的更加贴近实际使用的话，人型机器人还不够成熟。可以参考波士顿动力的人形机器人，纯粹实现了一个双足平衡和行走的能力，造价不菲。这种视频看看就好，想实际接触一下都非常困难。我们退而求其次可以选择国内有用的机器人，大概也就是NAO了。NAO有专门的SDK，机器人价格也不便宜，有渠道的话，8WRMB左右可以买到。NAO上装的舵机就要高档很多，配置也更加豪华。带陀螺仪，带超声波传感器，带压力传感器，带双摄，甚至可以带激光，带语音识别，带人脸检测。。。。功能非常丰富，当然全部加一起8WRMB还是有点贵。吊丝还是买优必选的玩玩，别的就算了吧。

目前我们已经可以在NAO上做基于camera的SLAM，功能虽然弱了一些，但是比自带的localization定位的要好太多了。