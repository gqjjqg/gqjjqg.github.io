---
layout: post
title: 竖直SeekBar
categories: [Development]
---

{{ page.title }}
================
　　今天做一个竖直方向的seekbar，本来想直接采用系统自带的seekabr更改风格来做，发现不太现实。参考网上常用的做法就是直接旋转画布，然后定制style背景和thumb。  
　　遇到一个兼容性的问题：就是Platform10的平台上，thumb始终有一半被遮挡。thumbOffset设置为0不起作用。但是在Platform11以上 设置为0则正常。为了兼容性只能另找它法，这里就不吐槽一下Android的碎片化问题了。  
　　最后的解决方案：设置topPadding和bottomPadding为thumb的半径，即可解决。过几天会放上 此定制的SeekBar。  
　　另外CheckBox 也有类似的问题，在Platform10和Platform13以上的左右边距 不同，导致check框和文字距离在不同版本的平台上有不同的表现。最后只能在values-v10和values-v13分别定义padding值，到时候一并上传。  
