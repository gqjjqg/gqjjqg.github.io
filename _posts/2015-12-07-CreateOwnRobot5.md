---
layout: post
title: Start create your own robot(5)
categories: [Development, Blog]
---

{{ page.title }}
================
经过上篇的模型验证和测试，现在可以把这个做的更加完美一些：设计一个扩展板，来真正替代alpha 1s的主控板：

<image src="http://gqjjqg.github.io/images/pcb.jpg" width="480" height="480" />

上位机则可以用A20的CubieBoard2板子来做一下性能的提升，顺带增配一个LCD触摸屏。

上面的PCB板电路可以重新设计一下，理论上alpha 1S的锂电池可以提供8V左右的电压，直接接入Nano的DC口供电，但是为了续航能力的考虑，还是采用外部5V供电比较好一些。上位机和LCD则需要增加一块5V2A的锂电池来供电，建议这个电池尺寸最好和IS的锂电池差不多大小，叠加放置在胸口位置比较好。这样既可以保持重心又不会占用额外的空间。

另外锂电池充电模块没有加入，可以考虑替换原来的DC充电口，增加供电模块。

<image src="http://gqjjqg.github.io/images/IMG_pcb.jpg" width="480" height="480" />

做出来的板子差不多就是这样，焊接上插座和开关，就可以直接上Nano和舵机的各个线路来使用了。





