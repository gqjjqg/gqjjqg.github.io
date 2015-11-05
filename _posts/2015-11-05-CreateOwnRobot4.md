---
layout: post
title: Start create your own robot(4)
categories: [Development, Blog]
---

{{ page.title }}
================
到目前为止，使用CubieBoard1已经基本验证，可以用这个开发板作为一个机器人的核心板。

作为轮式的移动平台，基本的简易功能都可以试做成功。

接下去就是玩人形机器了，市面上人形机器做的好的也就那么几家。优必选的alpha 1 大概5000RMB左右，炫酷的外形，巴拉巴拉。。。。

仔细分析这款机器人，[Alpha 1S](http://www.ubtrobot.com/html/archive/2015092894.html) 其实也就是一堆舵机的组合，并且被他们公司，用一个主控板限制了进一步的开发。为了更多的控制能力，可以直接买一块arduino 的 nano板，也就2x10厘米的大小。带2个串口。不带蓝牙功能，更省电！

准备好硬件，就开始动手拆分重组吧。

